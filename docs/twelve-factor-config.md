---
title: "Configuration, secrets, and feature flags"
status: proposed
version: 1.0.0
owner: "@Slight76"
---
# Configuration, secrets, and feature flags

Baseline: 1.0.0. Applies when: an application reads any setting that differs between environments or must stay out of source control

Decision: [ADR-0001](../adr/0001-adopt-operations-standards.md). Rules become binding when this baseline is adopted; examples explain the policy and do not establish business requirements.

This document applies the twelve-factor "config in the environment" principle to the team's stack: .NET APIs and workers, React frontends, Postgres, Docker images built by GitHub Actions and run on Fly.io. It refines PL-004 (secrets through approved management) and CICD-001 (one immutable artifact promoted across environments).

## What counts as configuration

Configuration is everything that varies between deployments of the same artifact: connection strings, external service URLs, credentials, log levels, feature toggles, resource budgets, and the public origin used in redirects. Code that behaves differently per environment because of `#if` blocks or hard-coded hostnames is a defect, not configuration.

| Category | Examples | Where it lives |
| --- | --- | --- |
| Non-secret defaults | Timeouts, page sizes, retry counts | `appsettings.json`, committed |
| Environment-specific, non-secret | Public API origin, log level, OTLP endpoint | `[env]` section of `fly.toml`, committed |
| Secret | Database URL, signing keys, third-party tokens | `fly secrets`, GitHub Actions secrets, user-secrets locally |
| Browser-visible | API base URL, analytics key, feature flags safe to expose | Public config artifact served with the frontend |

Anything in the browser bundle is public. A "secret" injected into a React build at `VITE_*`/`REACT_APP_*` time is published; treat it as such and keep real secrets server-side.

## One artifact, many environments

Build the Docker image once per commit and promote the same digest from staging to production. The image must contain no environment name, no connection string, and no `.env` file. Verify this with `docker history`/`docker run --rm <image> env` in CI; a secret visible in the image layer is a release blocker and a rotation event.

`.dockerignore` excludes `.env*`, `appsettings.*.json` that hold local overrides, `secrets.json`, and IDE folders. Multi-stage builds keep SDK-stage files out of the runtime image. Pin base images by digest.

## .NET configuration layering

Use the default host configuration order and do not reorder it: `appsettings.json` → `appsettings.{Environment}.json` → user secrets (Development only) → environment variables → command line. Environment variables win in every deployed environment. Map nested keys with `__` (`ConnectionStrings__Default`). Bind settings into typed options classes with `ValidateDataAnnotations().ValidateOnStart()` so a missing value fails at startup rather than on the first request.

Commit `appsettings.json` and `appsettings.Development.json` with safe local defaults only. Do not commit `appsettings.Production.json`; production differences come from the environment. `ASPNETCORE_ENVIRONMENT` selects the layer; a value other than `Development` disables developer exception pages and user secrets automatically, so never set production machines to `Development` for convenience.

## Fly.io secrets and environment

Non-secret values go in the `[env]` block of `fly.toml` and are reviewed in pull requests. Secrets are set with `fly secrets set KEY=value -a <app>` (or `fly secrets import` from a local file that is never committed) and appear in the VM only as environment variables. Setting a secret triggers a new deployment of the same image, which is the intended behaviour: configuration changes are deployments and show up in the release history.

Use a separate Fly app per environment (`myapp-staging`, `myapp`) rather than one app with branching logic. Each environment gets its own Postgres cluster and its own secrets; staging never holds production credentials. The Fly deploy token used by GitHub Actions is scoped to one app and stored as a repository or environment secret; environment-level secrets with required reviewers gate production.

## Local development

Developers use `dotnet user-secrets` for the API and an uncommitted `.env.local` for the frontend. Document every required key in `README.md` or `appsettings.json` with an empty or obviously fake default so a fresh clone fails fast with a readable message. Local Postgres runs in Docker Compose with throwaway credentials; those credentials are the only ones that may appear in committed files, and they must not work anywhere else.

## Feature flags

Feature flags are configuration with a lifecycle. Each flag has an owner, a purpose (release toggle, ops kill switch, or experiment), a default, and a removal date or condition. Read flags through `Microsoft.FeatureManagement` or a single typed options class, never by sprinkling `Environment.GetEnvironmentVariable` through the code. Release toggles are removed within two releases of being fully on; a flag older than that is a documented exception. Flags that change security behaviour (auth bypass, CORS origins, rate limits) are not flags; they are configuration reviewed under the security handbook.

## Verification

In CI, scan the built image for secret-like strings and fail on hits. In staging, start the app with one required secret removed and confirm it refuses to start with a message naming the key, not the value. Grep the frontend bundle for server-side keys. Review `fly secrets list` quarterly against the documented key inventory and remove orphans. Rotate any secret that ever appeared in a log, a PR, or an image.

Source: [The Twelve-Factor App, III. Config](https://12factor.net/config); [Fly.io secrets](https://fly.io/docs/apps/secrets/); [ASP.NET Core configuration](https://learn.microsoft.com/aspnet/core/fundamentals/configuration/).

## Rules and required evidence

| ID | Requirement | Verification |
| --- | --- | --- |
| CFG-001 | Deployable artifacts MUST be environment-agnostic; all environment-specific values MUST be supplied at runtime through the environment or platform secret store. | Image inspection and `fly.toml`/secrets review per environment |
| CFG-002 | Required configuration MUST be validated at startup, and secrets MUST NOT appear in images, committed files, browser bundles, or logs. | Startup failure test, bundle grep, secret scan in CI |
| CFG-003 | Feature flags MUST have an owner, purpose, default, and removal condition, and MUST be read through one typed access path. | Flag inventory review and code search |

## Exceptions

Use the [exception record](https://github.com/Slight76/standards-marketplace/blob/main/templates/exception.md) for a departure. Record affected rules, scope, compensating controls, approval evidence, expiry, and migration path. Agents must not silently replace defaults.

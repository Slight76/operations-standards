# operations-standards

Team operations standards for developers and AI agents: delivery pipelines, observability, SLOs and toil, incident response, configuration and secrets, platform and infrastructure (Fly.io, Docker, Postgres, GitHub Actions), backup and disaster recovery.

Part of the Slight76 standards handbooks indexed at [standards-marketplace](https://github.com/Slight76/standards-marketplace). Written for a small team and its AI agents.

## Documents

| Document | Covers | Rule prefixes |
| --- | --- | --- |
| [docs/platform-architecture.md](docs/platform-architecture.md) | Environments, artifact lifecycle, pipeline credentials, telemetry ownership, and secret handling at the platform level | PL |
| [docs/delivery-standard.md](docs/delivery-standard.md) | Build-once/promote pipelines, required CI stages, GitHub Actions security, deployment and rollback protocol | CICD |
| [docs/twelve-factor-config.md](docs/twelve-factor-config.md) | Config via environment, `appsettings` layering, Fly.io secrets, per-environment apps, image hygiene, feature flags | CFG |
| [docs/observability-standard.md](docs/observability-standard.md) | OpenTelemetry signals, cardinality and redaction, readiness/liveness semantics, SLO specification | OBS, SLO |
| [docs/slo-and-toil.md](docs/slo-and-toil.md) | Choosing SLIs per service, targets and error budgets, burn-rate alerting, toil budget and review cadence | TOIL |
| [docs/incident-response-and-postmortems.md](docs/incident-response-and-postmortems.md) | Severity levels, on-call expectations, response steps, blameless postmortem outline, action-item tracking | INC |
| [docs/backup-and-disaster-recovery.md](docs/backup-and-disaster-recovery.md) | Criticality tiers with RPO/RTO, backup inventory, Fly Postgres snapshot practice, restore drills, disaster scenarios | BDR |
| [docs/infrastructure-architecture.md](docs/infrastructure-architecture.md) | Compute, network, DNS, storage, failure domains, IaC state protection, recovery dependencies | INF |
| [docs/infrastructure-implementation-standard.md](docs/infrastructure-implementation-standard.md) | Platform selection, deployment inventory, trusted proxies and TLS, runtime hardening and capacity, IaC lifecycle | INF |
| [templates/operational-runbook.md](templates/operational-runbook.md) | Runbook skeleton: trigger and impact, diagnosis, recovery, verification, evidence | (used by INC, BDR, SLO) |

## Read by task

See [skills/operations-standards/SKILL.md](skills/operations-standards/SKILL.md).

## Install as an agent skill

| Agent | Command |
| --- | --- |
| Copilot CLI | `copilot plugin marketplace add Slight76/standards-marketplace` then `copilot plugin install operations-standards@slight76-standards` |
| GitHub CLI (any agent) | `gh skill install Slight76/operations-standards operations-standards --scope user --pin v1.0.0` |
| Claude Code | `/plugin marketplace add Slight76/standards-marketplace` then `/plugin install operations-standards@slight76-standards` |

## Layout

| Path | Purpose |
| --- | --- |
| `docs/` | Standards documents (frontmatter, applies-when, rule table) |
| `catalog/catalog.json` | Machine-readable rules; `externalDecisions` points at historic ADRs |
| `adr/` | Decisions local to this handbook |
| `skills/operations-standards/` | Agent skill and references |
| `templates/` | Templates specific to this domain (shared ones live in the marketplace) |

Validation: `py ../standards-marketplace/tooling/validate.py --root .`. License: [MIT](LICENSE).

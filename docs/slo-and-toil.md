---
title: "SLOs, error budgets, and toil"
status: proposed
version: 1.0.0
owner: "@Slight76"
---
# SLOs, error budgets, and toil

Baseline: 1.0.0. Applies when: a service runs in production and someone is expected to respond when it misbehaves

Decision: [ADR-0001](../adr/0001-adopt-operations-standards.md). Rules become binding when this baseline is adopted; examples explain the policy and do not establish business requirements.

[Observability, health, and service objectives](observability-standard.md) defines what an SLO specification must contain (SLO-001) and the signals it is built from (OBS-001, OBS-002). This document is the practice around it: how a small team picks objectives, spends an error budget, alerts on it without burning people out, and keeps operational drudgery within bounds.

## Choosing SLIs per service

Start from the user journey, not the infrastructure. For each production service, pick at most three SLIs; more than that dilutes attention.

| Service type | Default SLIs |
| --- | --- |
| HTTP API (.NET) | Availability: non-5xx responses / all eligible requests. Latency: requests under threshold / all eligible requests, at p95 or p99 by route group |
| Frontend (React, static) | Availability of the HTML/asset origin; largest contentful paint or time-to-interactive from real-user or synthetic checks |
| Background worker | Freshness: jobs completed within their deadline / all jobs. Correctness: jobs completed without dead-lettering / all jobs |
| Postgres (as a dependency) | Not an SLO of its own; its failures surface in the API and worker SLIs. Track saturation (connections, disk, replication lag) as alerts, not objectives |

Define the eligible denominator carefully: exclude health checks, bot traffic, and requests rejected with 4xx by design. Measure at the edge the user actually hits (Fly proxy or load balancer) where possible, and from the application when the edge cannot distinguish routes.

## Setting targets and windows

A target is a promise the team is willing to be woken for. Set it from observed behaviour over the last 30 days minus a margin, not from marketing. A new service starts with an aspirational target that is reviewed after one month of data. Use a rolling 30-day window for budgets and a 28-day or calendar-month window for reporting; state which.

Record the SLI query, target, window, owner, and exclusions in the service repository as a versioned file (for example `slo.yaml` or a section of the runbook). The dashboard is generated from it or reviewed against it; an SLO that exists only on a dashboard is not an SLO.

## Error budgets

Error budget = 1 − target, expressed as allowed bad events or bad minutes over the window. A 99.5% availability target over 30 days allows roughly 3.6 hours of full outage or the equivalent spread across partial failures.

Spend the budget deliberately: risky deploys, load tests in production, and dependency upgrades are the point of having one. When the remaining budget falls below 25% of the window, the team slows feature work on that service in favour of reliability until it recovers; when it is exhausted, only reliability fixes and reverts ship until the window rolls over. Write this policy down per service and have the owner sign it; the policy is what stops the budget from being a vanity metric.

## Alerting on burn rate

Page on the rate at which the budget is being consumed, not on a single threshold breach. Two windows catch both fast and slow burns without flapping:

| Alert | Burn rate | Long window | Short window | Action |
| --- | --- | --- | --- | --- |
| Fast burn | 14.4× (2% of 30-day budget in 1 hour) | 1 h | 5 min | Page the primary responder |
| Slow burn | 6× (5% of budget in 6 hours) | 6 h | 30 min | Page during waking hours, ticket otherwise |
| Trend | 1× (budget will be exhausted on schedule) | 3 d | 6 h | Ticket, review at next planning |

Both the long and short window must exceed the rate for the alert to fire, which stops a resolved incident from paging again. Every paging alert links a runbook built from the [operational runbook template](../templates/operational-runbook.md) and names the SLO it protects. Cause-based alerts (CPU high, pod restarted, queue long) are allowed as tickets or dashboard annotations; they do not page unless they are the only available proxy for user harm and say so in their description.

For services too small for statistically useful burn-rate math (under a few hundred requests per hour), fall back to a synthetic probe every minute and alert on consecutive failures; document the choice.

## Toil

Toil is manual, repetitive, automatable work tied to running a service that scales with its load: restarting a stuck worker, rotating a key by hand, running a report query for someone, re-queuing dead letters. It is not project work and it is not incident response.

The team sets a toil budget: operational toil above roughly 20% of an engineer's week over a month is a signal to stop and automate. Track toil candidly with a lightweight label on issues or a shared list; what is not written down cannot be reduced. Each recurring item gets one of three fates: automate it, eliminate the need for it, or accept it in writing with a review date. A runbook step performed more than three times a quarter is an automation candidate by default. Agents are a legitimate way to reduce toil for read-only diagnosis and report generation, within the authorization limits set elsewhere in these standards.

## Review cadence

Monthly: review each SLO's budget consumption, alert counts (paged, actionable, noisy), and toil list. Quarterly: revisit targets, retire alerts that never fired or never required action, and confirm runbooks match the current deployment. Record the review in the service repository.

## Verification

For each production service, show the versioned SLI definition, the dashboard that matches it, one fired burn-rate alert (real or exercised) with its runbook link, the written budget policy, and the current toil list with ages. A service with an SLO but no history of the budget influencing a decision is a candidate for a simpler target.

Source: [Google SRE Workbook, Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/); [Google SRE Book, Eliminating Toil](https://sre.google/sre-book/eliminating-toil/). Thresholds and budgets are team policy.

## Rules and required evidence

| ID | Requirement | Verification |
| --- | --- | --- |
| TOIL-001 | Teams MUST track recurring operational toil per service and MUST automate, eliminate, or explicitly accept each recurring item with an owner and review date. | Toil list review and automation evidence |

Objective, alert, and runbook requirements are governed by SLO-001 in [observability-standard.md](observability-standard.md); this document does not duplicate them.

## Exceptions

Use the [exception record](https://github.com/Slight76/standards-marketplace/blob/main/templates/exception.md) for a departure. Record affected rules, scope, compensating controls, approval evidence, expiry, and migration path. Agents must not silently replace defaults.

---
title: "Backup and disaster recovery"
status: proposed
version: 1.0.0
owner: "@Slight76"
---
# Backup and disaster recovery

Baseline: 1.0.0. Applies when: a service owns state that users or the business would miss if it were lost

Decision: [ADR-0001](../adr/0001-adopt-operations-standards.md). Rules become binding when this baseline is adopted; examples explain the policy and do not establish business requirements.

This document covers the operational side of recovery: targets per tier, what gets backed up, how often restores are proven, and what the runbook must contain. Database-level migration and recovery mechanics (point-in-time recovery, EF Core migration rollback, DR-001 and MIG-* rules) live in the data handbook's [migration and recovery standard](https://github.com/Slight76/data-standards/blob/main/docs/migration-recovery-standard.md). Infrastructure rebuild expectations come from INF-003 and INF-006.

## Criticality tiers and targets

Assign every service a tier from actual impact, then derive targets. Do not invent targets first and justify them later.

| Tier | Meaning | RPO (max data loss) | RTO (max time to restore) | Backup cadence |
| --- | --- | --- | --- | --- |
| 1 | Core user journey or money/obligation-bearing data | 15 minutes | 4 hours | Continuous WAL archiving plus daily snapshot |
| 2 | Important but recoverable; users tolerate a day of disruption | 24 hours | 1 working day | Daily snapshot |
| 3 | Internal, regenerable, or experimental | Best effort | Rebuild from source | Weekly snapshot or none, stated explicitly |

A service inherits the highest tier of any data it is the system of record for. Record the tier, RPO, RTO, and the evidence for the last restore in the service runbook. Targets that have never been measured are aspirations, and the runbook says so.

## What must be backed up

| Asset | Mechanism | Owner |
| --- | --- | --- |
| Postgres data | Platform snapshots plus logical dump (`pg_dump -Fc`) to object storage in a different provider or account | Service owner |
| Object storage (uploads, exports) | Versioning plus cross-bucket replication or scheduled sync | Service owner |
| Configuration and secrets inventory | `fly.toml` in git; secret *names* and their source documented; secret values held in a password manager with a break-glass procedure | Service owner |
| Container images | Registry retention covering every release still deployable; digest recorded per release | Delivery pipeline |
| Infrastructure definitions | Git; IaC state backed up and access-controlled | Infrastructure owner |
| DNS and certificates | Zone export in git; renewal automated and monitored | Infrastructure owner |

Code is not a backup target; the forge is. But the ability to rebuild from code depends on dependency availability, so lockfiles and, for tier 1, a mirror of critical packages or the last good image are part of the plan.

## Fly Postgres practice

Fly Postgres (both the unmanaged Fly Postgres app and Managed Postgres) provides volume snapshots with a default retention of five days. Treat snapshots as the fast path, not the only path: they live in the same region and account as the database. For tier 1 and tier 2:

- Extend snapshot retention to at least 14 days (`fly volumes snapshots` / `fly pg` settings) and record the retention in the runbook.
- Run a nightly `pg_dump -Fc` from a GitHub Actions scheduled workflow or a small Fly Machine, encrypt it, and upload it to object storage outside Fly with lifecycle rules (30 daily, 12 monthly). The workflow fails loudly, and a missed night is a ticket.
- Tier 1 services enable WAL archiving or Managed Postgres point-in-time recovery; document the achievable recovery window.
- Prefer a second replica in another region for tier 1 so a single-region outage is a failover, not a restore.
- Postgres connection strings for backup jobs use a read-only role; backup credentials never equal application credentials.

Restoring a snapshot creates a *new* cluster: the runbook must include updating the application's `DATABASE_URL` secret, redeploying, and reconciling anything written between the snapshot and the failure.

## Restore drills

A backup that has not been restored is a hope. Drill cadence by tier: tier 1 quarterly, tier 2 twice a year, tier 3 when convenient or never, stated explicitly. A drill restores into an isolated environment (a scratch Fly app and Postgres cluster), runs the application's smoke path against it, checks row counts and a few known records, measures elapsed time, and tears down. Record measured RPO and RTO against targets, every discrepancy, and the person who ran it in the runbook's evidence section. Automate the dump-and-restore verification where possible so the quarterly drill exercises the human steps rather than the mechanics.

Also drill the ugly cases at least once a year: restore with the primary engineer unavailable, restore when the Fly account itself is inaccessible (using the off-platform dump), and recover from a destructive migration applied to production.

## Disaster scenarios

Write down the response, in the runbook, for at least these: region outage, accidental `DROP`/bad migration, ransomware or account compromise, expired domain or certificate, loss of the only person who knows the secrets, and provider termination. Each entry names the trigger, the first three actions, the recovery path (failover, snapshot restore, off-platform restore, or full rebuild), who has authority to start it, and the expected duration. Account compromise response also follows the security handbook.

## Runbook linkage

Every tier 1 and tier 2 service has a recovery runbook built from the [operational runbook template](../templates/operational-runbook.md) and linked from the service `README.md`. It is tested, not just written: the last drill date and commit are in its header, and a runbook older than its tier's drill cadence is flagged in the monthly review described in [SLOs, error budgets, and toil](slo-and-toil.md). Incidents that invoke the runbook feed back corrections through the postmortem process in [incident response](incident-response-and-postmortems.md).

## Verification

For each service: tier and targets recorded; backup jobs visible and monitored for success; off-platform copy present and encrypted; last restore drill within cadence with measured RPO/RTO; runbook includes the secret rotation and `DATABASE_URL` update steps; at least two people have exercised the restore. A green backup dashboard without a dated restore record does not satisfy BDR-002.

Source: [Fly.io Postgres backup and restore](https://fly.io/docs/postgres/managing/backup-and-restore/); [PostgreSQL continuous archiving](https://www.postgresql.org/docs/current/continuous-archiving.html). Tiers and cadences are team policy.

## Rules and required evidence

| ID | Requirement | Verification |
| --- | --- | --- |
| BDR-001 | Every stateful service MUST be assigned a criticality tier with recorded RPO and RTO targets and a backup cadence that can meet them. | Tier record in the service runbook and backup job configuration |
| BDR-002 | Backups MUST be restored into an isolated environment on the tier's drill cadence, with measured RPO/RTO and discrepancies recorded. | Dated drill evidence in the runbook |
| BDR-003 | Tier 1 and tier 2 data MUST have an encrypted copy outside the primary hosting platform and account, with monitored backup jobs. | Off-platform storage listing and job alert history |

## Exceptions

Use the [exception record](https://github.com/Slight76/standards-marketplace/blob/main/templates/exception.md) for a departure. Record affected rules, scope, compensating controls, approval evidence, expiry, and migration path. Agents must not silently replace defaults.

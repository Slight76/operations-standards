# ADR-0001: Adopt the operations standards handbook

Status: Accepted

Date: 2026-10-07

Owner: @Slight76

## Context

The former single `architecture-standards` repository (v0.3.0) mixed every domain into one catalog and skill. Operational guidance (delivery pipelines, observability, platform and infrastructure architecture) sat beside solution-architecture and data rules, so an agent helping with a Fly.io deployment or an alert had to load everything. Incident response, configuration, SLO practice, and backup/DR had no home at all and were handled ad hoc per repository.

## Decision

Create `operations-standards` as the home for operations standards, versioned independently from the other handbooks and installable as an agent skill. Documents moved here keep their rule IDs and historical ADR references; new documents are first drafts with `status: proposed`.

New rules introduced by this handbook under this decision: CFG-001..003 (configuration and secrets), INC-001..003 (incident response and postmortems), TOIL-001 (toil tracking), and BDR-001..003 (backup and disaster recovery). They are `Proposed` until adopted by a consuming repository.

## Alternatives

- Keep the domain inside `architecture-standards`: rejected; one repository was too broad to read or install selectively.
- Rewrite all rules from scratch: rejected; existing rules are kept verbatim to preserve consumer baselines.
- Put incident and DR guidance in the security or data handbooks: rejected; they are run-time practices owned by whoever carries the pager, and they reference data and security rules rather than belong to them.

## Consequences

Consumers pin this repository in `architecture-baseline.json` (`standards[]`). Historic decisions remain in `architecture-standards/adr/` and are declared in `catalog/catalog.json` under `externalDecisions`. New rules in this handbook cite this ADR as their decision.

## Traceability

Rule prefixes: CICD, OBS, SLO, PL, INF, CFG, INC, TOIL, BDR. Related: standards-marketplace ADR-0001; architecture-standards ADR-0028..0031; data-standards `docs/migration-recovery-standard.md` for database recovery mechanics.

## Verification

`validate.py` passes; `docs.yml` green on `main`; the skill installs through the standards marketplace.

## Approval

@Slight76, 2026-10-07, plan approved in the split planning session.

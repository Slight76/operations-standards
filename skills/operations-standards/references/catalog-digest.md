# Catalog digest: operations-standards

Generated from `catalog/catalog.json` (version 1.0.0). Every rule ID with its statement, document, and decision.

| ID | Statement | Document | Decision | Status |
| --- | --- | --- | --- | --- |
| PL-001 | Application pipelines MUST independently build, test, and release immutable artifacts. | `docs/platform-architecture.md` | ADR-0007 | Proposed |
| PL-002 | Changes MUST run applicable unit, architecture, integration, contract, security, and critical journey checks. | `docs/platform-architecture.md` | ADR-0007 | Proposed |
| PL-003 | Production workloads MUST emit structured logs, useful metrics, traces, and readiness/liveness signals without exposing sensitive data. | `docs/platform-architecture.md` | ADR-0007 | Proposed |
| PL-004 | Secrets MUST be supplied through approved secret management and MUST NOT appear in source, images, or logs. | `docs/platform-architecture.md` | ADR-0007 | Proposed |
| INF-001 | Infrastructure MUST be version-controlled with reviewed plans and controlled production apply identities. | `docs/infrastructure-architecture.md` | ADR-0008 | Proposed |
| INF-002 | Data stores MUST use restricted network access and documented ingress/egress paths. | `docs/infrastructure-architecture.md` | ADR-0008 | Proposed |
| INF-003 | Production topology MUST document failure domains, capacity assumptions, DNS, certificates, and recovery dependencies. | `docs/infrastructure-architecture.md` | ADR-0008 | Proposed |
| CICD-001 | Pipelines MUST produce immutable artifacts and retain traceable build/test evidence. | `docs/delivery-standard.md` | ADR-0020 | Proposed |
| CICD-002 | Privileged workflows MUST isolate untrusted code and use least-privilege short-lived access where available. | `docs/delivery-standard.md` | ADR-0020 | Proposed |
| CICD-003 | Release plans MUST test client/server/schema compatibility and a concrete rollback path. | `docs/delivery-standard.md` | ADR-0020 | Proposed |
| OBS-001 | Production workloads MUST expose useful correlated signals with bounded cardinality and redaction. | `docs/observability-standard.md` | ADR-0021 | Proposed |
| OBS-002 | Readiness and liveness MUST have distinct bounded semantics and controlled exposure. | `docs/observability-standard.md` | ADR-0021 | Proposed |
| SLO-001 | Production services MUST define measurable user-centered objectives, owners, and actionable alerts. | `docs/observability-standard.md` | ADR-0021 | Proposed |
| INF-004 | Solutions MUST document and verify traffic boundaries, DNS/TLS ownership, and trusted proxy behavior. | `docs/infrastructure-implementation-standard.md` | ADR-0022 | Proposed |
| INF-005 | Runtime deployments MUST define resource limits, scaling dependencies, and graceful termination. | `docs/infrastructure-implementation-standard.md` | ADR-0022 | Proposed |
| INF-006 | Infrastructure state/configuration MUST be protected, reproducible, and reviewed for drift and destructive changes. | `docs/infrastructure-implementation-standard.md` | ADR-0022 | Proposed |
| CFG-001 | Deployable artifacts MUST be environment-agnostic; all environment-specific values MUST be supplied at runtime through the environment or platform secret store. | `docs/twelve-factor-config.md` | ADR-0001 | Proposed |
| CFG-002 | Required configuration MUST be validated at startup, and secrets MUST NOT appear in images, committed files, browser bundles, or logs. | `docs/twelve-factor-config.md` | ADR-0001 | Proposed |
| CFG-003 | Feature flags MUST have an owner, purpose, default, and removal condition, and MUST be read through one typed access path. | `docs/twelve-factor-config.md` | ADR-0001 | Proposed |
| INC-001 | Production services MUST have a named primary responder, a fallback, and documented severity levels with response and communication targets. | `docs/incident-response-and-postmortems.md` | ADR-0001 | Proposed |
| INC-002 | Incidents MUST be recorded with a timeline, actions taken, and user impact from declaration to close, and recovery MUST prefer tested rollback over live debugging. | `docs/incident-response-and-postmortems.md` | ADR-0001 | Proposed |
| INC-003 | SEV1 and SEV2 incidents MUST produce a blameless postmortem within five working days with owned, tracked action items. | `docs/incident-response-and-postmortems.md` | ADR-0001 | Proposed |
| TOIL-001 | Teams MUST track recurring operational toil per service and MUST automate, eliminate, or explicitly accept each recurring item with an owner and review date. | `docs/slo-and-toil.md` | ADR-0001 | Proposed |
| BDR-001 | Every stateful service MUST be assigned a criticality tier with recorded RPO and RTO targets and a backup cadence that can meet them. | `docs/backup-and-disaster-recovery.md` | ADR-0001 | Proposed |
| BDR-002 | Backups MUST be restored into an isolated environment on the tier's drill cadence, with measured RPO/RTO and discrepancies recorded. | `docs/backup-and-disaster-recovery.md` | ADR-0001 | Proposed |
| BDR-003 | Tier 1 and tier 2 data MUST have an encrypted copy outside the primary hosting platform and account, with monitored backup jobs. | `docs/backup-and-disaster-recovery.md` | ADR-0001 | Proposed |

Rule count: 26. Prefixes: BDR, CFG, CICD, INC, INF, OBS, PL, SLO, TOIL.

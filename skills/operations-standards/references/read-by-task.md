# Read by task: operations-standards

Full task-to-document map for this handbook. Paths are relative to the repository root. Rule prefixes tell you which IDs to cite in evidence.

| Task or question | Read first | Then | Rule prefixes |
| --- | --- | --- | --- |
| Add or change a GitHub Actions workflow | `docs/delivery-standard.md` | security-standards `docs/application-security-standard.md` for supply chain | CICD, PL |
| Define required CI checks and merge gates | `docs/delivery-standard.md` | `docs/platform-architecture.md` | CICD, PL |
| Write or change a Dockerfile / image build | `docs/delivery-standard.md` | `docs/infrastructure-implementation-standard.md` (runtime hardening) | CICD, INF, CFG |
| Deploy to Fly.io (`fly.toml`, `fly deploy`, machines, regions) | `docs/infrastructure-implementation-standard.md` | `docs/delivery-standard.md` (deployment protocol) | INF, CICD |
| Plan a rollback or verify schema/client compatibility | `docs/delivery-standard.md` | data-standards `docs/migration-recovery-standard.md` | CICD |
| Environment variables, `appsettings.*.json`, user secrets | `docs/twelve-factor-config.md` | `docs/platform-architecture.md` (PL-004) | CFG, PL |
| Fly secrets, GitHub Actions secrets, secret rotation | `docs/twelve-factor-config.md` | security-standards secrets guidance | CFG, PL |
| Add a feature flag or remove an old one | `docs/twelve-factor-config.md` | | CFG |
| Structured logging, redaction, log retention | `docs/observability-standard.md` | | OBS, PL |
| Metrics, labels, cardinality, dashboards | `docs/observability-standard.md` | `docs/slo-and-toil.md` | OBS |
| Distributed tracing / OpenTelemetry setup | `docs/observability-standard.md` | | OBS |
| Health endpoints (`/alive`, `/health`), readiness vs liveness | `docs/observability-standard.md` | `docs/infrastructure-implementation-standard.md` (graceful shutdown) | OBS, INF |
| Define an SLI/SLO for a service | `docs/slo-and-toil.md` | `docs/observability-standard.md` (SLO specification) | SLO, TOIL |
| Write an alert or decide whether it should page | `docs/slo-and-toil.md` | `docs/incident-response-and-postmortems.md` (on-call) | SLO, INC |
| Error budget policy, slowing feature work | `docs/slo-and-toil.md` | | SLO |
| Track or reduce toil, automate a runbook step | `docs/slo-and-toil.md` | `templates/operational-runbook.md` | TOIL |
| An incident is happening right now | `docs/incident-response-and-postmortems.md` (response steps) | the service runbook | INC |
| Write a postmortem or track action items | `docs/incident-response-and-postmortems.md` | | INC |
| Set up on-call or severity levels | `docs/incident-response-and-postmortems.md` | | INC |
| Write or review an operational runbook | `templates/operational-runbook.md` | `docs/backup-and-disaster-recovery.md`, `docs/incident-response-and-postmortems.md` | INC, BDR |
| Decide backup cadence, RPO/RTO, criticality tier | `docs/backup-and-disaster-recovery.md` | | BDR |
| Fly Postgres snapshots, off-platform dumps | `docs/backup-and-disaster-recovery.md` | data-standards `docs/migration-recovery-standard.md` | BDR |
| Run or record a restore drill | `docs/backup-and-disaster-recovery.md` | `templates/operational-runbook.md` | BDR |
| Disaster scenarios, region loss, account compromise | `docs/backup-and-disaster-recovery.md` | security-standards incident/disclosure guidance | BDR, INC, INF |
| Network boundaries, ingress/egress, trusted proxies | `docs/infrastructure-implementation-standard.md` | `docs/infrastructure-architecture.md` | INF |
| DNS, TLS certificates, custom domains | `docs/infrastructure-implementation-standard.md` | `docs/backup-and-disaster-recovery.md` (zone export) | INF, BDR |
| Resource limits, scaling, graceful termination | `docs/infrastructure-implementation-standard.md` | | INF |
| Infrastructure as code, state, drift | `docs/infrastructure-implementation-standard.md` | `docs/infrastructure-architecture.md` | INF |
| Choosing a platform (managed, containers, VMs, Kubernetes) | `docs/infrastructure-implementation-standard.md` | `docs/infrastructure-architecture.md` | INF |
| Environment separation, artifact lifecycle, telemetry ownership | `docs/platform-architecture.md` | `docs/delivery-standard.md`, `docs/observability-standard.md` | PL |
| Departing from any default | marketplace `templates/exception.md` | the document owning the rule | any |

## Cross-handbook pointers

| Need | Handbook |
| --- | --- |
| Schema migrations, EF Core, point-in-time recovery, DR-001 | [data-standards](https://github.com/Slight76/data-standards/blob/main/docs/migration-recovery-standard.md) |
| Authentication, CORS, secrets policy, threat modelling, SBOM | [security-standards](https://github.com/Slight76/security-standards) |
| Testing levels and evidence | [engineering-standards testing standard](https://github.com/Slight76/engineering-standards/blob/main/docs/testing-standard.md) |
| Adopting a baseline in a repository | [engineering-standards adoption process](https://github.com/Slight76/engineering-standards/blob/main/docs/adoption-process.md) |
| Governance, ownership, exceptions | [standards-marketplace](https://github.com/Slight76/standards-marketplace) |

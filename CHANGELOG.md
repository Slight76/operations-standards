# Changelog

All notable changes to this handbook. Versions follow SemVer; consumers pin commit SHAs in `architecture-baseline.json`.

## 1.0.1 - 2026-10-07

- Declare `license: MIT` in the skill frontmatter so `gh skill publish` validates cleanly.
- Pin the shared docs-lint workflow to a marketplace commit SHA.

## 1.0.0 - 2026-10-07

First release as a standalone handbook, split from `Slight76/architecture-standards@c1bda3d` (v0.3.0). See [ADR-0001](adr/0001-adopt-operations-standards.md).

### Moved from architecture-standards (rule IDs and external ADR references preserved)

| Document | Former path | Rules |
| --- | --- | --- |
| `docs/platform-architecture.md` | `platform/architecture.md` | PL-001..PL-004 |
| `docs/delivery-standard.md` | `platform/delivery-standard.md` | CICD-001..CICD-003 |
| `docs/observability-standard.md` | `platform/observability-standard.md` | OBS-001, OBS-002, SLO-001 |
| `docs/infrastructure-architecture.md` | `infrastructure/architecture.md` | INF-001..INF-003 |
| `docs/infrastructure-implementation-standard.md` | `infrastructure/implementation-standard.md` | INF-004..INF-006 |
| `templates/operational-runbook.md` | `templates/operational-runbook.md` | - |

Wording was de-enterprised (team standards, no corporate-governance layer) and links rewritten to the split handbooks.

### New documents (status: proposed)

| Document | Rules |
| --- | --- |
| `docs/twelve-factor-config.md` | CFG-001..CFG-003 |
| `docs/incident-response-and-postmortems.md` | INC-001..INC-003 |
| `docs/slo-and-toil.md` | TOIL-001 |
| `docs/backup-and-disaster-recovery.md` | BDR-001..BDR-003 |

### Added

- `adr/0001-adopt-operations-standards.md` (decision for the split and for the new rules above).
- `skills/operations-standards/` agent skill with `references/read-by-task.md` and generated `references/catalog-digest.md`.
- `catalog/catalog.json` with 26 rules and `externalDecisions` for ADR-0007, ADR-0008, ADR-0020, ADR-0021, ADR-0022.
- Shared tooling configuration (`.markdownlint-cli2.jsonc`, `lychee.toml`, `.github/workflows/docs.yml`) from the marketplace scaffold.

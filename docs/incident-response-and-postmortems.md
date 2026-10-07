---
title: "Incident response and blameless postmortems"
status: proposed
version: 1.0.0
owner: "@Slight76"
---
# Incident response and blameless postmortems

Baseline: 1.0.0. Applies when: a production service is degraded, unavailable, leaking data, or at imminent risk of any of those

Decision: [ADR-0001](../adr/0001-adopt-operations-standards.md). Rules become binding when this baseline is adopted; examples explain the policy and do not establish business requirements.

This is an incident process sized for a small team: one or two people respond, there is no dedicated incident commander rotation, and the same engineers who build the service run it. The goal is fast, calm recovery and a written record that makes the next incident less likely. It relies on the alerts and runbooks required by SLO-001 and the [operational runbook template](../templates/operational-runbook.md).

## Severity levels

Declare severity from user impact, not from how the problem looks internally. Over-declare and downgrade later; the reverse costs more.

| Severity | User impact | Response | Communication |
| --- | --- | --- | --- |
| SEV1 | Core journey down for most users, data loss or exposure suspected, or security breach | Respond immediately, any hour; one person on recovery, one on comms if available | Status update within 15 minutes, then every 30 minutes until mitigated |
| SEV2 | Core journey degraded (errors, slowness) or secondary feature down for many users | Respond within 30 minutes during waking hours, next morning otherwise unless burn rate says sooner | Status update within 1 hour, then every 2 hours |
| SEV3 | Minor feature broken, workaround exists, or internal tooling affected | Next working day | Note in the team channel; no customer notice unless asked |

Anything suspected to involve unauthorized access or personal data is SEV1 until shown otherwise and follows the security handbook's disclosure obligations in addition to this process.

## On-call expectations

With a small team, "on-call" means a named primary for the week and a documented fallback, both recorded where everyone can see them. Expectations are explicit: acknowledge SEV1 pages within 15 minutes, carry a device that can run `fly` and reach the dashboards, and hand over with a written note of anything in progress. Nobody is on call during booked leave; if coverage is impossible, say so in writing rather than silently degrading the promise.

Only alerts tied to user harm page. Everything else goes to a channel and is reviewed during working hours. An alert that pages twice without needing action is fixed or deleted the same week; see [SLOs, error budgets, and toil](slo-and-toil.md).

## Response steps

1. **Acknowledge and declare.** Ack the alert, post `INCIDENT SEV<n>: <one line>` in the incident channel, and open an incident issue from the template. Record the start time as the first user-visible impact if known, otherwise the alert time.
2. **Stabilize first.** Prefer the fastest safe action from the runbook: roll back to the last known-good release (`fly releases` then `fly deploy --image <previous digest>`), scale up, disable a feature flag, or fail over. Do not debug root cause while users are down if a rollback is available. Rollbacks must respect schema compatibility (CICD-003); never roll back code past an irreversible migration without the data owner.
3. **Communicate.** One person owns updates. Each update says what users see, what is being done, and when the next update comes. Avoid speculation about cause.
4. **Preserve evidence.** Before restarting or redeploying, capture logs, traces, `fly status`, and the release ID. Note every command run, with timestamps, in the incident issue as you go; this becomes the postmortem timeline.
5. **Mitigate, then monitor.** Declare mitigated when the SLI returns to normal for a defined period (default 30 minutes for SEV1). Keep the incident open until the monitoring period passes.
6. **Close and schedule the postmortem.** Record end time, final severity, and customer-facing summary. Schedule the postmortem within five working days for SEV1/SEV2.

Agents assisting during an incident operate under the same authorization limits as at any other time: read-only diagnosis is encouraged, and any production-changing command is proposed to a human, not executed autonomously.

## Blameless postmortem

Every SEV1 and SEV2 gets a written postmortem; SEV3s get one when someone learns something worth sharing. The document describes what the system and the people did with the information they had, and never names a person as the cause. "The deploy pipeline allowed an untested migration to reach production" is a finding; "X deployed without testing" is not.

Template outline (store under `docs/postmortems/` in the consuming repository, named `YYYY-MM-DD-short-title.md`):

1. **Summary**: one paragraph, impact in user terms, duration, severity.
2. **Impact**: users affected, requests failed, data affected, SLO/error-budget consumed, revenue or obligation impact if known.
3. **Timeline**: timestamped entries from first change or trigger through detection, response actions, mitigation, and resolution. Include time-to-detect and time-to-mitigate.
4. **Root cause and contributing factors**: the technical chain, plus the process or tooling gaps that let it happen and let it persist.
5. **What went well / what went poorly / where we got lucky**: three short lists.
6. **Action items**: each with owner, due date, tracking issue, and type (prevent, detect faster, mitigate faster, process).
7. **Lessons**: what changes in how the team builds or runs things.

Review the draft together within a week; the meeting exists to improve the analysis and agree on actions, not to assign fault. Publish it where the whole team can read it.

## Action-item tracking

Action items are issues in the service's repository labelled `postmortem`, linked from the postmortem, and reviewed in the regular planning cadence. Prefer a small number of items that will actually be done over an exhaustive wish list. Items overdue by more than one cycle are either rescheduled with a written reason or closed as "won't do" with the accepted risk noted in the postmortem. Repeat incidents with an open action item from a previous postmortem are automatically at least SEV2 in the review, whatever the user impact, because the system failed to learn.

## Verification

Run one tabletop exercise per quarter: pick a plausible failure (database out of disk, bad migration, expired certificate, leaked token), walk the runbook, and time the steps. Confirm the primary can roll back from a laptop without asking anyone for credentials. Check that the incident issue template and postmortem template exist in each production repository. Review open `postmortem` issues monthly.

Source: [Google SRE Book, Postmortem Culture](https://sre.google/sre-book/postmortem-culture/); [PagerDuty Incident Response](https://response.pagerduty.com/). Severity levels and timings are team policy.

## Rules and required evidence

| ID | Requirement | Verification |
| --- | --- | --- |
| INC-001 | Production services MUST have a named primary responder, a fallback, and documented severity levels with response and communication targets. | On-call record and severity table in the service runbook |
| INC-002 | Incidents MUST be recorded with a timeline, actions taken, and user impact from declaration to close, and recovery MUST prefer tested rollback over live debugging. | Incident issue review and rollback exercise evidence |
| INC-003 | SEV1 and SEV2 incidents MUST produce a blameless postmortem within five working days with owned, tracked action items. | Postmortem document and linked issues |

## Exceptions

Use the [exception record](https://github.com/Slight76/standards-marketplace/blob/main/templates/exception.md) for a departure. Record affected rules, scope, compensating controls, approval evidence, expiry, and migration path. Agents must not silently replace defaults.

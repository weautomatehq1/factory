---
file: RUNBOOK.md
status: skeleton
populated_when: first incident occurs OR M-018 daily digest exposes a recurring alert (rule of two — second occurrence triggers entry)
last_updated: 2026-05-26
owner: sebas
template_source: Google SRE Book chs. 11-16
---

# Runbook

> **Status: skeleton.** This file is intentionally near-empty. We do not pre-write runbook entries for incidents that have not happened — those become stale before they're needed. Per Google SRE practice, runbook entries are added when an alert fires (and crucially, on the *second* time it fires without an existing entry).
>
> When entries are added, they MUST follow the structure below.

## Per-entry template (use this format for every entry)

```markdown
## <Alert / Failure name>
status: active | retired
last_verified: YYYY-MM-DD by <name> — <pass | fail | partial>

### Symptoms
- What the alert text says
- User-visible behavior (if any)
- Dashboards / graphs to check first
- Discord channel where the alert appeared

### Diagnosis
- Decision tree, top-down
- Exact commands / queries / dashboard URLs
- Expected outputs and what each means
- Common false-positive patterns

### Remediation
- Ordered steps, idempotent where possible
- Rollback command for each step
- "If step N fails, go to step M"
- Time-box: if not resolved in X minutes, escalate

### Escalation
- When to page Incident Commander (Sebastian)
- When to involve Ops Lead vs Comms Lead vs Planning Lead (Google SRE roles)
- External contact (vendor support line, status page URL)

### Verification
- How to confirm fix took hold
- SLO/SLI to watch for next 30 minutes
- When to declare incident closed

### Related
- Postmortems: [link]
- ADRs: [link]
- Risk register entries: [R-NNN]

### Last verified
- YYYY-MM-DD by <name> — <outcome>
```

## Index of entries

_(Empty. Entries will be added here as runbook items are written.)_

## Trigger rules — when an entry MUST be added

Per Google SRE Book Ch. 15 "Postmortem Culture":

1. **User-visible downtime or degradation past threshold** (TBD per-product SLOs)
2. **Any data loss event**, no matter how small
3. **On-call manual intervention** (rollback, reroute, killswitch toggle)
4. **Resolution exceeded time limit** (TBD per-alert)
5. **Monitoring failure** (we couldn't see what was happening)
6. **Stakeholder request** (client, Sebastian, Esme asks for a runbook entry)
7. **Rule of two:** any alert that fires a second time without an existing runbook entry

## Escalation matrix (until real entries exist, use this)

| Severity | Who to ping | Where | SLA |
|---|---|---|---|
| Sev 1: data loss, security breach, production down | Sebastian | Discord DM + phone | Immediate |
| Sev 2: customer-visible degradation, billing fail | Sebastian | Discord #ifleet @mention | <30 min |
| Sev 3: internal degradation, single worker stuck | Sebastian | Discord #ifleet | <4 hours |
| Sev 4: cosmetic, non-blocking | none | GitHub Issue with `severity:4` label | Next business day |

## Postmortem template (use after every Sev 1 / Sev 2)

Per Google SRE blameless postmortem format. Lives at `docs/postmortems/YYYY-MM-DD-<incident-name>.md`.

Sections: Summary · Impact · Root Causes · Trigger · Resolution · Detection · Action Items (with owners and deadlines) · Lessons Learned (What Went Well / What Went Poorly / Where We Got Lucky) · Timeline.

---

## Tier-1 self-heal escalation (specific to The Factory)

When IFleet's Tier 1 (build-time) intervention agent fails 3 consecutive times on the same issue, Discord alert posts:

```
[Tier 1 escalation] Issue #NNN, worker <worker-id>, 3 failed attempts.
Last error: <error-trunc>
Diagnosis attempted: <diagnosis-trunc>
Action needed: human review.
```

Operator response: read `memory/handoff.md` in the worktree, decide whether to (a) fix the spec, (b) fix the worker prompt, (c) close issue as out-of-scope, (d) re-queue with hints.

## Tier-2 self-heal escalation

When auto-rollback fires:

```
[Tier 2 rollback] PR #NNN, deploy <deploy-id>, rolled back at <time>.
Smoke test failure: <test-name>
Sentry breadcrumb: <link>
Auto-issue: #NNN created.
```

Operator response: review the auto-issue, decide whether the rollback was correct (sometimes it wasn't — false positive smoke).

## Tier-3 self-heal escalation (when M-017 lands)

When auto-merge gate blocks (any gate fails):

```
[Tier 3 gate blocked] PR #NNN proposed fix for Sentry issue <id>.
Blocked by: <gate-name>
Path touched: <path>
Manual merge required.
```

Operator response: review the draft PR, merge if appropriate.

---

**Last updated:** 2026-05-26
**Last verified:** 2026-05-26 — nightly audit (read-only review; SLO gap flagged as AUDIT-factory-c5e702e1)

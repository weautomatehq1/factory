---
file: KPI.md
status: skeleton
populated_when: M-016 (real client #1) ships AND first 7 days of production data captured
last_updated: 2026-05-15
owner: sebas
frameworks: ["Sean Ellis North Star", "HEART (Rodden CHI 2010)", "Doerr OKRs"]
---

# KPIs — The Factory

> **Status: skeleton.** Metrics MUST be backed by real data. We do not invent baseline numbers. This file gets populated when client #1 has 7 days of production data and we can capture a baseline.
>
> Self-healing Tier 3 depends on this file having real `alert_threshold` values — until they're populated against real data, Tier 3 stays off.

## Filling rule

Every metric row MUST have:
- `name` — single canonical name (snake_case)
- `definition` — one-sentence plain English
- `type` — `leading` or `lagging`
- `target` — numeric, with timeframe (e.g., `1200/week by 2026-Q3`)
- `query` — SQL string OR analytics URL the agent can hit
- `alert_threshold` — drop-trigger as % vs trailing N-week average (e.g., `-15% vs trailing 4-week avg`)
- `last_baselined` — date

Without those fields, **self-healing has nothing to evaluate**. A row without `query` and `alert_threshold` is a placeholder, not a metric.

---

## North Star Metric (single, leading)

Per Sean Ellis: must be a leading indicator. Revenue is lagging — don't pick it.

Candidates to validate with real data (DO NOT lock until client #1 has data):

- `weekly_active_clients_with_completed_action` — clients whose users hit the critical-path action at least 3 times that week
- `time_from_interview_end_to_spec_pr_open` — operational KPI for The Factory itself (target: <30 min)
- `percent_features_shipped_without_human_architect_loopback` — measures spec-quality (target: >70%)

**Placeholder North Star (until baselined):**
```yaml
name: <TBD — populate from M-016>
definition: <TBD>
type: leading
target: <TBD>
query: <TBD>
alert_threshold: <TBD>
last_baselined: <pending — no data yet>
```

---

## Supporting Metrics (L2)

### Leading (agent-controllable, react in hours)

- `voice_interview_completion_rate` — % of started interviews that reach `end_interview()`
- `spec_coverage_score_at_interview_end` — average coverage % across required sections
- `worker_pickup_latency_seconds` — time from issue open to worker claim
- `pr_open_to_merge_hours` — median time PR sits awaiting human review
- `ci_pass_rate_first_attempt` — proxy for spec/code quality

### Lagging (business outcomes, react in weeks)

- `client_satisfaction_score` — NPS or similar (per-client survey)
- `time_from_first_call_to_client_signoff` — full-cycle time
- `cost_per_completed_client_project` — total IFleet + voice spend / completed projects
- `monthly_recurring_revenue_per_client` — when applicable

---

## HEART Pillars (Rodden et al., CHI 2010)

One table per per-client SaaS once that client's KPI.md is populated. The Factory itself uses these only for the operator-facing dashboard.

| Pillar | Goal | Signal | Metric | Query |
|---|---|---|---|---|
| Happiness | <TBD per client> | <TBD> | NPS, CSAT | <TBD> |
| Engagement | <TBD per client> | <TBD> | sessions/user/wk, key actions/session | <TBD> |
| Adoption | <TBD per client> | <TBD> | new workspaces activated weekly | <TBD> |
| Retention | <TBD per client> | <TBD> | W4 retention | <TBD> |
| Task Success | <TBD per client> | <TBD> | critical-path completion rate | <TBD> |

---

## OKRs (current quarter)

Per Doerr's grammar: "I will [Objective] as measured by [Key Results]"

### Q2 2026 (until 2026-06-30)

> ⚠️ **OKR update required (2026-05-26 nightly audit; updated 2026-06-06):** KR1 deadline (2026-05-20) passed — SPRINT-2026-05-A ended incomplete (zero feature commits). KR2 deadline (2026-05-27) passed — SPRINT-2026-05-B has not been opened. KR3 deadline (2026-06-10) is 4 days away — effectively missed: depends on KR2 (client #1 interview), which has no active sprint and no scheduled date. KR4 deadline (2026-06-04) passed — M-018 not built, no active sprint (MISSED). These OKRs must be revised by Sebastian/Esme in the next sprint planning session. Current status preserved for audit trail.

**Objective:** Operationalize The Factory and onboard client #1 end-to-end.

**Key Results:** (numeric, time-bound — populate as we hit milestones)
- KR1: Complete SPRINT-2026-05-A dry-run with all 15 milestones green — DUE 2026-05-20 ❌ MISSED (sprint incomplete)
- KR2: Onboard client #1 via voice interview → merged spec PR — DUE 2026-05-27 (at-risk — no active sprint)
- KR3: Client #1's phase 1 staging URL reviewed and signed-off by client — DUE 2026-06-10 ⚠️ EFFECTIVELY MISSED (depends on KR2 which is itself blocked; no client scheduled; 4 days remain)
- KR4: First Discord 8am digest contains at least 5 days of real data — DUE 2026-06-04 ❌ MISSED (M-018 not built; no active sprint)

**Confidence (1-10):**
- KR1: 0 (missed — SPRINT-2026-05-A confirmed incomplete as of 2026-05-21)
- KR2: 2 (no active sprint; Retell R1 research still pending; client not scheduled)
- KR3: 1 (depends on KR2 which is blocked; deadline 2026-06-10 effectively missed)
- KR4: 0 (deadline passed 2026-06-04; M-018 not built; no active sprint)

---

## Anti-metrics (things we do NOT measure)

- **Lines of code shipped per worker** — incentivizes verbosity, not quality
- **Number of PRs merged per day** — incentivizes splitting/rushing
- **Time-to-first-commit** — see above
- **Worker uptime** — workers are spawned per task; uptime is irrelevant
- **Cost per PR** — useful directionally but optimizes the wrong thing (a $5 PR that ships a real feature beats a $0.50 PR that doesn't)

---

## Review cadence

| Cadence | Item | Owner |
|---|---|---|
| Daily (8am digest) | Leading metrics (last 24h) | M-018 automation |
| Weekly | All metrics; baseline re-check | Sebastian |
| Monthly | OKR confidence re-score | Sebastian |
| Per postmortem | Detection-time metric (D-score in RISKS.md) | Sebastian |

---

**Last updated:** 2026-06-06
**Last verified:** 2026-06-06 — nightly audit (KR3 marked effectively missed; closes AUDIT-factory-20260606-01)

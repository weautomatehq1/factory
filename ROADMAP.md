---
file: ROADMAP.md
status: living
last_updated: 2026-06-17
last_reconciled: 2026-06-17
owner: sebas
---

# Roadmap

> Now / Next / Later / Not Doing. Milestones are bracketed `[M-NNN]` so IFleet's decomposer can regex-match them. `depends-on` builds the DAG that gates worker pickup.

> ⚠️ **Reconciliation note (2026-06-17):** Prior reconciliation paragraphs are superseded. Factory git reality 2026-05-23 → 2026-06-10 shows 8 nightly-audit cycles closing ~73 findings — real delivery, just not roadmap-milestone-shaped. IFleet shipped M5 substrate (#323/#325/#326), M5.2 partial (Approve→/ship enqueue + PR-outcome tracking merged), M6 substrate with flag-gated closure paths (`DRIFT_REAL_PR`/`BANDIT_LIVE` default OFF), M4.5 Phase C routing migration (IFleet ADR-0004 — see IFleet docs/adr/0004-canonical-routing-alignment.md, not to be confused with factory ADR-0004 which covers pipeline-first strategy), and an audit-hardening sweep (#374-#386, 15 findings incl. 6 CRITICAL security; closeouts #387-#390). None of this advances M-002/3/4 (the IFleet "spec decomposer / DAG queue / policy hook" gaps), which remain genuinely planned and blocked on IFleet sprint capacity, not on M-001. [M-001] spec-template repo flipped from `deferred` to `shipped`. [M-005] voice-discovery repo does not exist — `planned` is correct. SPRINT-2026-05-B was never opened; the current de-facto sprint is IFleet's M5.2/M6 push tracked in `~/dev/IFleet/SPRINT.md`, not in this file.

---

## Shipped (out-of-cycle delivery)

### [M-NEW-A] Nightly self-audit cadence (factory)
- status: shipped
- delivered: 2026-05-23 → present
- evidence: ~73 findings closed across 8 nightly-audit cycles. See `git log --grep "nightly audit"`.
- owner: sebas

### [M-NEW-B] IFleet audit-hardening sweep 2026-06
- status: shipped
- delivered: 2026-06 (PRs #374-#386, 15 findings, 6 CRITICAL security; closeouts #387-#390)
- evidence: PRs #374-#390 in `weautomatehq1/IFleet`
- cross-link: `~/dev/IFleet/docs/runbooks/audit-hardening-2026-06.md`
- owner: sebas

### [M-NEW-C] M5 Proposer substrate live (IFleet)
- status: shipped (substrate), gated (live deploy on `#ifleet-proposals` channel + `PROPOSER_ENABLED=1`)
- delivered: 2026-06 (IFleet PRs #323/#325/#326)
- owner: sebas

---

## Now (current cycle — re-plan pending in SPRINT-2026-05-B; SPRINT-2026-05-A ran 2026-05-15 → 2026-05-20, incomplete)

### [M-001] Spec-template repo scaffolded
- status: shipped
- shipped_date: 2026-05-16
- evidence: `gh repo view weautomatehq1/spec-template` — repo created 2026-05-16, `isTemplate: true`
- follow-up: verify success criteria #2 (17 files with skeleton frontmatter) and #3 (README orients) in follow-up.
- owner: sebas
- appetite: 1 day
- depends-on: []
- success-criteria:
  - GitHub repo `weautomatehq1/spec-template` created as a template repo
  - 17 spec files present with `status: skeleton` frontmatter for placeholders
  - `README.md` orients a new user
  - "Use this template" button produces a working clone
- sprint: SPRINT-2026-05-A
- adrs: [ADR-0001, ADR-0002]

### [M-002] IFleet Gap 1 — Spec decomposer worker
- status: planned
- owner: esme
- appetite: 1 day
- depends-on: [M-001]
- success-criteria:
  - Worker reads `ROADMAP.md` + `ARCHITECTURE.md` from a merged spec PR
  - Emits dependency-ordered GitHub issues, each with `spec: <file>#<anchor>` reference
  - Respects `NON_GOALS.md` (no out-of-scope issues)
  - Respects `SECURITY.md` (flags protected-path issues for human review)
- sprint: SPRINT-2026-05-A
- adrs: [ADR-0003]

### [M-003] IFleet Gap 2 — Dependency-aware queue
- status: planned
- owner: esme
- appetite: 0.5 day
- depends-on: [M-002]
- success-criteria:
  - Workers pick next *unblocked* issue (DAG-aware)
  - Issue dependency declared via `depends-on:` body field
  - Stuck-on-dep issues are visible in `.omc/STATUS.md`
- sprint: SPRINT-2026-05-A
- adrs: []

### [M-004] IFleet Gap 5 — Cross-cutting policy hook
- status: planned
- owner: esme
- appetite: 0.5 day
- depends-on: [M-001]
- success-criteria:
  - Pre-merge hook re-reads `AGENTS.md` and flags drift (naming, error-handling, auth patterns)
  - Hookify-based, defined in markdown
  - Blocks merge if violation detected; opens issue otherwise
- sprint: SPRINT-2026-05-A
- adrs: []

### [M-005] Voice-discovery repo + Retell + n8n + Supabase wiring
- status: planned
- owner: sebas
- appetite: 1 day
- depends-on: []
- success-criteria:
  - GitHub repo `weautomatehq1/voice-discovery` created
  - Retell account active with one phone number
  - n8n workflow: Retell webhook → Supabase persist transcript
  - Test call ends → row in Supabase with full transcript
- sprint: SPRINT-2026-05-A
- adrs: [ADR-0006]

### [M-006] Voice interviewer brain (Layer 1 + Layer 2)
- status: planned
- owner: sebas
- appetite: 1 day
- depends-on: [M-005]
- success-criteria:
  - System prompt encodes Layer 1 (therapy mode) + Layer 2 (coverage tracker)
  - 4 tools wired: `mark_covered`, `flag_ambiguity`, `escalate_to_human`, `end_interview`
  - 24-question library (Mom Test + JTBD + Strategyzer) loaded
  - Test interview reaches coverage ≥ 80% on required sections
- sprint: SPRINT-2026-05-A
- adrs: []

### [M-007] Operator dashboard
- status: planned
- owner: sebas
- appetite: 0.5 day
- depends-on: [M-005]
- success-criteria:
  - Next.js page on weautomatehq.cloud
  - Live transcript stream (Retell webhook)
  - Coverage heatmap (filled vs missing spec sections, confidence per)
  - "Interrupt" button — Sebastian joins live mid-call
- sprint: SPRINT-2026-05-A
- adrs: []

### [M-008] IFleet Gap 4 — Schema-first migration worker
- status: planned
- owner: esme
- appetite: 0.5 day
- depends-on: [M-002, M-003]
- success-criteria:
  - Reads `UBIQUITOUS_LANGUAGE.md` + `ARCHITECTURE.md`
  - Emits SQL migrations + RLS policies
  - Migrations land in `db/migrations/` with timestamp naming
- sprint: SPRINT-2026-05-A
- adrs: []

### [M-009] IFleet Gap 6 — Design system bootstrap
- status: planned
- owner: esme
- appetite: 0.5 day
- depends-on: [M-002, M-003]
- success-criteria:
  - One-shot worker reads brand intent from INTAKE
  - Outputs Tailwind config + base shadcn/ui components
  - Uses `awesome-design-md` skill from `~/.claude/skills/`
- sprint: SPRINT-2026-05-A
- adrs: []

### [M-010] Spec synthesizer
- status: planned
- owner: sebas
- appetite: 1 day
- depends-on: [M-001, M-006]
- success-criteria:
  - Claude Opus pipeline reads transcript + coverage map from Supabase
  - Generates all 17 spec files from `spec-template`
  - Creates new GitHub repo for client
  - Opens PR #1 on `feat/initial-spec` branch
  - End-to-end: call ends → PR #1 open within 30 min
- sprint: SPRINT-2026-05-A
- adrs: []

### [M-011] Self-healing Tier 1 + Tier 2 wired
- status: planned
- owner: sebas
- appetite: 1 day
- depends-on: [M-002, M-003]
- success-criteria:
  - Tier 1: CI failure → DoVer intervention agent → retry → escalate
  - Tier 2: post-deploy smoke fail → rollback in <60s → Sentry breadcrumb → auto-issue
  - Killswitches `SELF_HEAL_BUILD=off`, `SELF_HEAL_DEPLOY=off` honored
  - All actions logged to `~/.omc/heal-log/<date>.jsonl`
- sprint: SPRINT-2026-05-A
- adrs: [ADR-0005]

### [M-012] IFleet Gap 7 — Deploy worker + smoke harness
- status: planned
- owner: esme
- appetite: 0.5 day
- depends-on: [M-002]
- success-criteria:
  - Phase complete → Vercel staging deploy triggered
  - Playwright smoke tests run against staging
  - Pass = staging URL posted to Discord #ifleet; fail = M-011 Tier 2 fires
- sprint: SPRINT-2026-05-A
- adrs: []

### [M-013] IFleet Gap 8 — Client-visible staging URL
- status: planned
- owner: esme
- appetite: 0.25 day
- depends-on: [M-012]
- success-criteria:
  - Every PR auto-deploys to Vercel preview
  - Comment with URL posted to PR
  - "What's done" view embedded in PR description
- sprint: SPRINT-2026-05-A
- adrs: []

### [M-014] IFleet Gap 3 — Project bootstrapper (OAuth-heavy)
- status: planned
- owner: sebas
- appetite: 1 day
- depends-on: []
- success-criteria:
  - n8n workflow (idempotent) creates: Supabase project + GitHub repo + GitHub secrets + Vercel project + Stripe webhook + Sentry project + PostHog project
  - DNS records configured via Hostinger API
  - Bootstrap completes in <10 min
- sprint: SPRINT-2026-05-A
- adrs: []

### [M-015] Dry-run on fake client interview
- status: planned
- owner: sebas+esme
- appetite: 0.5 day
- depends-on: [M-001, M-002, M-003, M-004, M-005, M-006, M-007, M-008, M-009, M-010, M-011, M-012, M-013, M-014]
- success-criteria:
  - Sebastian plays fake client; voice interviewer runs the call
  - Synthesizer generates 17 spec files
  - PR #1 reviewed and merged
  - IFleet decomposer creates issues
  - Workers ship at least one throwaway feature (e.g., `GET /healthz`)
- sprint: SPRINT-2026-05-A
- adrs: []

---

## Next (next cycle — 2026-05-21 → 2026-06-04; window expired with zero delivery; re-plan pending)

### [M-016] Real client #1 — interview through full pipeline
- status: planned
- depends-on: [M-015]
- appetite: 2 weeks
- success-criteria:
  - Real client onboarded via voice interview
  - Spec PR #1 reviewed + merged
  - Phase 1 of their SaaS shipped to staging
  - Client signs off on phase 1 staging URL

### [M-017] Self-healing Tier 3 — runtime (gated rollout)
- status: planned
- depends-on: [M-016, +30 days production data]
- appetite: 1 week
- success-criteria:
  - Sentry event / PostHog anomaly → IFleet opens DRAFT PR with proposed fix
  - Auto-merge ONLY if: CI green + screenshot diff match + risk score < threshold + outside business hours + change < 50 LOC + path not in `SECURITY.md` Protected Paths
  - Discord ping for manual merge otherwise
  - Killswitch `SELF_HEAL_RUNTIME=off`

### [M-018] Discord 8am digest
- status: planned
- depends-on: [M-015]
- success-criteria:
  - Daily digest at 8am to #ifleet
  - Contents: PRs merged overnight, errors caught, costs, KPI deltas, self-heals fired
  - Sebastian + Esme see the same digest

### [M-019] Cost-tracking dashboard
- status: planned
- depends-on: [M-014]
- success-criteria:
  - Per-client costs visible in `.omc/costs.json` aggregated to a dashboard
  - Daily rate cap enforced ($10/day normal, unlimited in `ralph`/`autopilot`)

---

## Later (this quarter — 2026-Q2 / Q3)

### [M-020] Client #2 and #3 onboarded
- Validates that the pipeline scales beyond client #1.

### [M-021] Esme's claude-toolkit merger
- Per memory: Esme's separate updated claude-toolkit eventually merges with this.

### [M-022] Per-client dashboards
- Each client gets a status URL where they see their build progress live.

### [M-023] Multi-language interview support
- Spanish, then Portuguese. Retell supports both.

### [M-024] Custom branding per sub-account
- White-label option for agencies who resell.

---

## Not Doing (explicit no-gos)

> **Canonical source is `NON_GOALS.md`.** Do not maintain a duplicate list here — that creates sync drift. The 10 No-Gos (NoGo-1 through NoGo-10) are defined there. If you're a worker checking whether a task is in-scope: check `NON_GOALS.md`, not this section.

---

## Open questions

- **Q1:** Self-heal autonomy level for client #1 (Option B `auto-merge low-risk` recommended). Decision pending.
- **Q2:** ~~Spec-template location confirmation (own repo `weautomatehq1/spec-template` recommended).~~ **Resolved by Q1** — repo exists at `weautomatehq1/spec-template`, `isTemplate: true`, created 2026-05-16. [M-001] flipped to `shipped`.
- **Q3:** 🚨 **BLOCKING DECISION — OVERDUE (33+ days):** Voice platform commitment (Retell default vs. Esme R1). SPRINT-2026-05-A ended 2026-05-20; as of 2026-06-22 (33 days elapsed) no confirmation or pivot recommendation has landed. SPRINT-2026-05-B has not been created. Sebastian must set a hard deadline. M-005 through M-007, M-010, M-015 are all blocked on this decision. Until confirmed, Retell remains default per ADR-0011 (supersedes ADR-0006).

---

**Last updated:** 2026-06-20
**Last verified:** 2026-06-17 — manual reconciliation against factory git log + IFleet PR history

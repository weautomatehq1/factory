---
file: ROADMAP.md
status: living
last_updated: 2026-06-13
last_reconciled: 2026-06-07
owner: sebas
---

# Roadmap

> Now / Next / Later / Not Doing. Milestones are bracketed `[M-NNN]` so IFleet's decomposer can regex-match them. `depends-on` builds the DAG that gates worker pickup.

> ⚠️ **Reconciliation note (2026-05-21):** SPRINT-2026-05-A ended 2026-05-20 but `git log --since=2026-05-15` on factory returns only the scaffold commit (`d49b1a4`). Statuses below have been rolled back to reflect git reality — anything previously implying progress in this repo has been changed to `deferred` or `not-started` with reason. Sibling repos (IFleet, voice-discovery) are NOT backfilled into this file by audit-fix `AUDIT-factory-b5bd1864`. See SPRINT.md §Reconciliation note for full context. A new SPRINT-2026-05-B will re-plan based on current reality.

## Now (current cycle — re-plan pending in SPRINT-2026-05-B; SPRINT-2026-05-A ran 2026-05-15 → 2026-05-20, incomplete)

### [M-001] Spec-template repo scaffolded
- status: deferred
- reason: no factory commits since 2026-05-15 scaffold; spec-template repo state not tracked here
- previous_status: in-progress (was incorrectly carried forward from 2026-05-15)
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

> ⚠️ **Reconciliation note (2026-06-04 nightly audit):** The "next cycle" window (2026-05-21 → 2026-06-04) ended today with none of milestones [M-016] through [M-019] delivered. SPRINT-2026-05-B was never created — there is no active sprint document. Statuses below remain `planned`. All work in this section is deferred until SPRINT-2026-05-B opens. Sebastian and Esme must define the new sprint before any milestone can progress. See `.omc/STATUS.md` "Up next" for immediate blockers.

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
- **Q2:** Spec-template location confirmation (own repo `weautomatehq1/spec-template` recommended). Decision: A — confirmed implicitly by directory creation 2026-05-15.
- **Q3:** Voice platform commitment (Retell default, Esme R1 research pending — see [M-005]). ⚠️ **ACTION REQUIRED**: Esme R1 confirmation is overdue. SPRINT-2026-05-A ended 2026-05-20; as of 2026-06-10 (21 days elapsed) no confirmation or pivot recommendation has landed. SPRINT-2026-05-B has not been created. Sebastian must set a hard deadline. Until confirmed, Retell remains default per ADR-0011 (supersedes ADR-0006).

---

**Last updated:** 2026-06-13
**Last verified:** 2026-06-13 — nightly audit (clean scan, date maintenance)

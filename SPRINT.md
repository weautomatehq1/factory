---
sprint: SPRINT-2026-05-A
appetite: 5 days
starts: 2026-05-15
ends: 2026-05-20
milestones: [M-001, M-002, M-003, M-004, M-005, M-006, M-007, M-008, M-009, M-010, M-011, M-012, M-013, M-014, M-015]
owner: sebas
co_owner: esme
status: active
---

# Sprint: The Factory — full pipeline build

## Problem

We have a real paying SaaS client waiting. Without The Factory operational, we'd hand-craft the client's project the way Sebastian has done before — slow, context-lossy, and not repeatable. The opportunity cost of skipping the pipeline build is one client at a time, forever. With it: every future client onboards through a voice interview → spec → autonomous build, and the operator's day becomes "review PRs + approve gates."

Per ADR-0004, we chose the proper-build-first path over the manual-shortcut-for-client-#1 path. Manual shortcuts always leave debt.

## Appetite

**5 days. Hard stop.** If not done by 2026-05-20 EOD, we cut scope per the priority order in §Tasks below — we do not extend.

## Solution sketch

Two parallel tracks for 5 days, converging on a dry-run.

**Sebastian track (voice/spec side):**
- D1: spec-template repo scaffolded with 17 file skeletons (M-001)
- D2: voice-discovery repo + Retell + n8n + Supabase wiring (M-005)
- D3: voice interviewer brain (M-006) + operator dashboard (M-007)
- D4: spec synthesizer (M-010) + project bootstrapper (M-014) + self-healing T1/T2 (M-011)
- D5: dry-run + fix

**Esme track (IFleet evolution):**
- D1: spec decomposer (M-002)
- D2: dependency-aware queue (M-003) + policy hook (M-004)
- D3: schema worker (M-008) + design bootstrap (M-009)
- D4: deploy worker (M-012) + staging URL (M-013)
- D5: dry-run + fix

**Convergence (D5):** Sebastian plays fake client. Voice interviewer runs the call. Synthesizer generates spec. IFleet decomposer creates issues. Workers ship one throwaway feature end-to-end. Both fix what breaks.

```mermaid
gantt
    title 5-day build (parallel tracks)
    dateFormat YYYY-MM-DD
    axisFormat %a %d
    section Sebas (voice/spec)
    M-001 spec-template      :done, m001, 2026-05-15, 1d
    M-005 voice infra        :m005, 2026-05-16, 1d
    M-006 interviewer brain  :m006, after m005, 1d
    M-007 operator dash      :m007, after m005, 1d
    M-010 synthesizer        :m010, 2026-05-18, 1d
    M-014 bootstrapper       :m014, 2026-05-18, 1d
    M-011 self-heal T1/T2    :m011, 2026-05-18, 1d
    section Esme (IFleet)
    M-002 decomposer         :m002, 2026-05-15, 1d
    M-003 DAG queue          :m003, after m002, 0.5d
    M-004 policy hook        :m004, after m002, 0.5d
    M-008 schema worker      :m008, 2026-05-17, 0.5d
    M-009 design bootstrap   :m009, 2026-05-17, 0.5d
    M-012 deploy worker      :m012, 2026-05-18, 0.5d
    M-013 staging URL        :m013, after m012, 0.25d
    section Both
    M-015 dry-run            :m015, 2026-05-19, 1d
    Real client interview    :2026-05-20, 1d
```

## Rabbit holes (known traps to avoid)

- **Retell tool-calling edge cases** — interrupts during a tool call, mid-call state loss. **Time-box debug to 4 hours**, then fall back to "agent calls tools only between turns" if quirky.
- **n8n + Retell webhook auth quirks** — first integration is always slower. **Use n8n's built-in webhook auth, not custom HMAC**, until proven insufficient.
- **Bootstrapper provider OAuth flows** — Hostinger, Vercel, Stripe — each new provider is a hidden afternoon. **Build OAuth one provider at a time, not all in parallel.**
- **Spec synthesizer prompt iteration** — first generation will be bad. **Plan 3–5 prompt iterations.** Don't ship the first one.
- **Self-heal Tier 2 rollback testing** — easy to under-cover. **Intentionally break something on D5 to test** rather than waiting for production.
- **Multi-repo coordination overhead** — opening PRs across 3 repos for one feature gets sloppy. **Per-day sync at EOD** to make sure both tracks are aligned.

## No-gos (explicit, this sprint)

- **NOT building Tier 3 runtime self-healing.** No production data yet to tune gates against. Deferred to M-017.
- **NOT building per-client dashboards** beyond a minimal staging URL. Deferred to M-022.
- **NOT optimizing IFleet worker cost.** Default routing (sonnet/opus per complexity label) is fine for the sprint.
- **NOT writing comprehensive tests for new IFleet workers in this sprint.** Smoke tests only. Full coverage in next cycle. Trade-off documented.
- **NOT migrating from Retell if R1 research suggests Vapi is marginally cheaper.** Switching mid-sprint = sprint death. Locked for the sprint.

## Tasks (decomposer fills this — manually seeded for SPRINT-A)

Priority order (cut from bottom if appetite slips):

- [ ] [M-001] spec-template scaffolded — sebas, D1
- [ ] [M-002] IFleet decomposer — esme, D1
- [ ] [M-005] voice-discovery + Retell + n8n + Supabase — sebas, D2
- [ ] [M-003] dependency-aware queue — esme, D2
- [ ] [M-004] policy hook — esme, D2
- [ ] [M-006] voice interviewer brain — sebas, D3
- [ ] [M-007] operator dashboard — sebas, D3 (CUT if D3 slips)
- [ ] [M-008] schema worker — esme, D3
- [ ] [M-009] design bootstrap — esme, D3 (CUT if D3 slips)
- [ ] [M-010] spec synthesizer — sebas, D4
- [ ] [M-011] self-heal T1/T2 — sebas, D4
- [ ] [M-012] deploy worker — esme, D4
- [ ] [M-014] bootstrapper — sebas, D4 (CUT to D5 if D4 slips)
- [ ] [M-013] staging URL — esme, D4
- [ ] [M-015] dry-run — both, D5

## Done when

- All M-001 through M-015 success-criteria pass (see ROADMAP.md for per-milestone criteria)
- Sebastian and Esme both sign off on the D5 dry-run video/recording
- An ADR is written for any architectural decision made during the sprint
- Discord 8am digest at least once shows real data
- This file's `status` flips to `shipped` and a new SPRINT-2026-05-B is opened

---

**Last updated:** 2026-05-15
**Last verified:** 2026-05-15 — Sebastian

---
adr: "0004"
title: "Build full pipeline before client #1 (no manual shortcuts)"
status: accepted
date: 2026-05-15
deciders: [sebas]
consulted: [esme]
informed: [ifleet, voice-discovery]
tags: [strategy, sprint]
supersedes: null
---

# ADR-0004: Build full pipeline before client #1

## Context and Problem Statement

A real paying SaaS client is waiting. We could either (a) build a quick voice interviewer + synthesize the spec manually + build the client's product by hand with Claude Code (manual shortcut), or (b) build the entire pipeline first (5 days), then onboard the client through it. Which strategy?

## Decision Drivers

- Client #1 is real and waiting
- Sebastian and Esme work fast and do not need urgency-compressed plans
- Manual shortcuts always leave debt — would have to be re-done properly later
- Pipeline matures faster on real-world signal than on theoretical edge cases
- 5 days of focused work is meaningfully shorter than open-ended "build it later"
- Sebastian's own working style ("not just cool I built it — structured the correct way") rejects shortcut paths

## Considered Options

1. **Manual shortcut** — quick voice interview (3 days), then Sebastian + Claude Code hand-build client #1, harden the pipeline in background over weeks
2. **Full pipeline first** — 5 days both Sebastian and Esme parallel-building all components, then real client #1 onboarded through finished system
3. **Half-pipeline** — voice + spec automated, IFleet evolution deferred

## Decision Outcome

Chosen: **Option 2 — full pipeline first.**

5-day parallel sprint (SPRINT-2026-05-A):
- D1–D5: 15 milestones executed across 2 tracks (Sebastian voice/spec; Esme IFleet evolution)
- D6+: client #1 onboarded through the finished system

### Why not the others?
- **Option 1 (manual shortcut):** every manual step we'd take for client #1 = work we'd have to throw away when we automate it later. Worse, the manual outputs would set bad precedents (e.g., "the first INTAKE was hand-curated, so the synthesizer doesn't need to match this quality"). Building right the first time is cheaper.
- **Option 3 (half-pipeline):** voice interview producing a spec that nothing can build with is half a system. Pointless.

## Consequences

- **Good:**
  - Client #1 onboards through the finished system → first deliverable is also our dogfood validation
  - No tech debt from manual workarounds
  - Pipeline gets real-world signal immediately
  - Sebastian's "correct structure" criterion satisfied
- **Bad:**
  - Client waits 5–10 days for the system to be ready before onboarding
  - 5-day sprint is ambitious — risk of slip (RISKS R-005, R-007, R-008)
  - Both Sebastian and Esme must be available for the full 5 days
- **Neutral:**
  - We accept the upfront time investment for a sustainable pace

## Confirmation

- Verify: D5 dry-run succeeds end-to-end (Sebastian plays fake client → voice → spec → IFleet builds throwaway feature)
- Verify: client #1 onboarded by D10 (soft) or D14 (hard ceiling)
- Re-evaluate after client #1: was 5-day-build worth the wait? Or would a 3-day-quick-build + client #1 have been better?

## Related

- Related: SPRINT.md (current sprint definition), ROADMAP [M-001] through [M-015]
- Source: this Sebastian-Claude conversation 2026-05-15, "we are in no rush" message

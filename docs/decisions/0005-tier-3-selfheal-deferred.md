---
adr: "0005"
title: "Tier 3 runtime self-healing deferred until 2 weeks production data"
status: accepted
date: 2026-05-15
deciders: [sebas]
consulted: [esme, claude-orchestrator, ops-research]
informed: [ifleet]
tags: [self-healing, safety]
supersedes: null
---

# ADR-0005: Tier 3 runtime self-healing deferred

## Context and Problem Statement

The Factory's self-healing system has three tiers: Tier 1 (build-time), Tier 2 (deploy-time), Tier 3 (runtime — Sentry/PostHog → auto-PR with proposed fix). Tier 3's auto-merge gate uses thresholds (risk score, screenshot diff sensitivity, business-hours window, change-size). Should we ship Tier 3 in the initial pipeline build or defer it?

## Decision Drivers

- Auto-merge thresholds must be calibrated against real data, not invented
- A miscalibrated Tier 3 = false auto-merges = customer-visible bad fixes
- Tier 1 + Tier 2 are mostly already designed (DoVer intervention agent + rollback hook); cost to ship them now is low
- Tier 3 design is sound but unproven without real Sentry/PostHog signal
- Sebastian's working principle: correct structure > clever output

## Considered Options

1. **Ship all three tiers in SPRINT-2026-05-A** — Tier 3 with conservative invented thresholds
2. **Ship Tier 1 + Tier 2 now, Tier 3 in next cycle** — defer Tier 3 until production data exists to tune against
3. **Skip self-healing entirely until client #1 is live** — minimal automation now

## Decision Outcome

Chosen: **Option 2 — Tier 1 + Tier 2 in SPRINT-2026-05-A, Tier 3 deferred to M-017 (after client #1 has 2 weeks of clean production data).**

Tier 3 stays off by default (`SELF_HEAL_RUNTIME=off`) until M-017 ships.

### Why not the others?
- **Option 1 (ship all three):** invented thresholds are exactly the "cool not correct" pattern Sebastian explicitly rejects. We'd ship a system whose safety properties we can't prove until weeks of data — by which point bad auto-merges may have happened.
- **Option 3 (skip self-healing):** wastes the design work for Tier 1 + Tier 2 (which are largely ready), gives operators no safety net during the most fragile phase (early build).

## Consequences

- **Good:**
  - Tier 3's gates calibrated against real signal (RISKS R-010 mitigation)
  - Operators have build/deploy safety net immediately
  - Tier 3 ships once with high confidence, not iteratively with mistakes
- **Bad:**
  - First 2+ weeks of production have no auto-fix for runtime issues — operator pages on Sentry events instead
  - Slight risk Tier 3 design rots before M-017 — mitigation: revisit ARCHITECTURE.md cross-cutting section at M-016 acceptance
- **Neutral:**
  - Killswitch `SELF_HEAL_RUNTIME` exists from day 1 so the gate is plumbed even when off

## Confirmation

- Verify: Tier 1 + Tier 2 fire correctly during SPRINT-2026-05-A dry-run (M-015)
- Verify: at M-016 + 14 days, baseline data is captured to calibrate Tier 3 thresholds
- Verify: at M-017, auto-merge gate has measured-not-invented thresholds documented in M-017's ADR
- Audit: zero false auto-merges in the first 30 days after M-017 ships

## Related

- Related: SECURITY.md "Killswitches"; AGENTS.md "Boundaries" Always-list
- Related milestones: [M-011] (Tier 1+2), [M-017] (Tier 3)
- Related risks: R-002 (auto-merge breaking change), R-008 (rollback failure), R-010 (KPI regression undetected)

---
adr: "0010"
title: "Issues created by 4 upstream sources + humans"
status: accepted
date: 2026-05-15
deciders: [sebas]
consulted: [esme]
informed: [ifleet, voice-discovery]
tags: [ifleet, self-healing]
supersedes: null
---

# ADR-0010: Issues from four upstream sources + humans

## Context and Problem Statement

ADR-0003 established that GitHub Issues remain the unit of work. But who creates them? Today: only humans. Going forward: multiple automated sources will also create them. We need to enumerate the sources, agree on their roles, and prevent confusion.

## Decision Drivers

- Every source of work should produce an Issue (per ADR-0003)
- Workers shouldn't have to know which upstream source created an issue — they all look the same
- Operators need to be able to filter / triage by source (so we can see "what did self-heal queue overnight?")
- New upstream sources should not require new IFleet code (extensible by label, not by code path)

## Considered Options

1. **One source only (humans)** — keep IFleet's current trigger model
2. **Two sources (humans + Decomposer)** — automate spec → issue, keep everything else manual
3. **Four upstream sources + humans** — Decomposer, Sentry/PostHog bridge, Deploy worker, Cron monitor; plus humans

## Decision Outcome

Chosen: **Option 3 — four upstream sources + humans.**

The five paths:

| Source | What triggers it | What it creates |
|---|---|---|
| **Decomposer** | Spec PR merge | N issues (one per milestone or sub-task), dependency-ordered |
| **Sentry/PostHog bridge** | Sentry error / PostHog anomaly | 1 issue with stack trace + recent commits |
| **Deploy worker** | Failed post-deploy smoke test | 1 issue with deploy-id, failed test name, rollback status |
| **Cron monitor** | Failed scheduled job | 1 issue with job name, last successful run, error log |
| **Humans** | Any time | 1 issue (manual file via GitHub UI) |

Every auto-created issue is labeled with its source: `source:decomposer`, `source:sentry`, `source:deploy`, `source:cron`. Workers don't filter on this; operators do.

### Why not the others?
- **Option 1 (humans only):** misses the whole point of self-healing (Sentry events become silent unless humans triage). Also misses the Decomposer (which is M-002, already planned).
- **Option 2 (humans + Decomposer):** captures the build-time gap but leaves runtime + ops bare. Then we'd have to retro-add bridges later, which means re-deciding contracts.

## Consequences

- **Good:**
  - One queue, one worker contract — all sources interchangeable to workers
  - Operators can filter by source label for triage
  - Adding a new source (e.g., security scanner) doesn't require IFleet code changes — just a new label
  - Sentry → n8n → GitHub bridge already exists per [[project-status-20260513]] — this ADR formalizes the contract
- **Bad:**
  - Auto-issue volume can be high. Mitigation: dependency-aware queue (M-003) prevents bulk pickup; rate limits per source label.
  - Bad upstream sources could spam issues (e.g., a noisy Sentry alert). Mitigation: each source has a circuit breaker — if it creates >N issues per hour, pause that source and alert operator.
- **Neutral:**
  - We may add a 5th source later (e.g., security scanner). When we do, write an addendum ADR or supersede this one.

## Confirmation

- Verify (M-002): Decomposer creates issues with `source:decomposer` label
- Verify (after M-017): Sentry events produce `source:sentry` labeled issues via n8n bridge
- Verify: operator can filter the Issues page by source
- Audit: at 90 days, what's the source-distribution? Are any sources dominating? Tune accordingly.

## Related

- Related: ADR-0003 (Issues as unit of work)
- Related: ARCHITECTURE.md "External systems" + `#5-architectural-invariants` (item 3)
- Related: ROADMAP [M-002] (Decomposer); existing Sentry bridge per [[project-status-20260513]]

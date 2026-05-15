---
adr: "NNNN"
title: "Short noun phrase"
status: proposed
date: YYYY-MM-DD
deciders: [sebas]
consulted: [esme, claude-orchestrator]
informed: [ifleet, voice-discovery]
tags: [tag1, tag2]
supersedes: null
---

# ADR-NNNN: <Short noun phrase>

## Context and Problem Statement

2–3 sentences. What forces are at play? Frame as a question if useful.

## Decision Drivers

- Driver 1 (e.g., "must support multi-tenant from day one")
- Driver 2 (e.g., "must work for both Claude and Codex workers")
- Driver 3

## Considered Options

1. **Option A** — short description
2. **Option B** — short description
3. **Option C** — short description (if any)

## Decision Outcome

Chosen: **Option B**, because <one-sentence justification tied to drivers>.

### Why not the others?
- Option A: why rejected (cite driver)
- Option C: why rejected (cite driver)

## Consequences

- **Good:**
  - Consequence A
  - Consequence B
- **Bad:**
  - Consequence C (tradeoff accepted because…)
- **Neutral:**
  - Consequence D

## Confirmation

How we'll verify this decision held up:
- Test, metric, or audit cadence
- Specific evidence we'd look for to know we got it wrong

## Related

- Supersedes: ADR-XXXX (if any)
- Related ARCHITECTURE.md section: `#section-anchor`
- Related risks: R-NNN
- Related milestones: [M-NNN]

---

**Notes for the author:**
- Filename: `NNNN-kebab-title.md` (sequential, never reused)
- Status: start as `proposed`; flip to `accepted` after sign-off
- Never edit an `accepted` ADR — supersede with a new one
- Cite primary sources in body when relevant

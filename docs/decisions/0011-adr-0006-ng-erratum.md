---
adr: "0011"
title: "ADR-0006 erratum: clarify NG-reference for mid-sprint platform lock"
status: accepted
date: 2026-05-29
deciders: [sebas]
consulted: [claude-nightly-audit]
informed: [voice-discovery, ifleet]
tags: [voice, non-goals, erratum]
supersedes: null
---

# ADR-0011: ADR-0006 erratum — clarify NG-reference for mid-sprint platform lock

## Context and Problem Statement

ADR-0006 (Retell AI as voice platform default) contains the phrase:

> _"The pick is locked for the sprint (per NON_GOALS.md NG-something equivalent — switching mid-sprint = sprint death)."_

The phrase `NG-something equivalent` is a lazy placeholder from the initial ADR draft and does not resolve to a real NON_GOALS.md entry. Since ADR-0006 is `status: accepted` and is append-only, it cannot be edited in-place. This erratum clarifies the intended reference. (AUDIT-factory-dd92db4f.)

## Decision Drivers

- ADR append-only invariant prevents correcting the placeholder in ADR-0006 directly
- Workers reading ADR-0006 cannot verify the NG claim without a real ID
- The underlying intent (no mid-sprint platform pivot) is architecturally correct and should be preserved

## Considered Options

1. **Add erratum ADR (this option)** — append-only-compliant; clarifies the reference without touching the accepted ADR
2. **Supersede ADR-0006** — would re-open the Retell platform decision unnecessarily; overkill for a reference fix
3. **Edit ADR-0006 directly** — violates the append-only invariant enforced by pre-commit hook; rejected

## Decision Outcome

Chosen: **Option 1 — erratum ADR.**

The placeholder `NG-something equivalent` in ADR-0006's Decision Outcome section refers to the general spirit of **NON_GOALS.md §No-Gos rule: avoid switching integration platforms mid-sprint**. This is currently captured implicitly by the combination of:

- **NON_GOALS.md NG-6:** "No gold-plating; implement what the spec says, not what seems cooler"
- **AGENTS.md §7 "Never":** "Edit a `status: accepted` ADR" (the same append-only discipline that prevents thrashing)
- **SPRINT.md scope-lock principle:** once a sprint goal is committed, pivots require a new sprint, not an in-flight swap

Until NON_GOALS.md gains an explicit NG entry for platform stability within a sprint, workers should treat the ADR-0006 lock as grounded in the above three rules collectively.

### Why not the others?

- Option 2: superseding ADR-0006 forces a full re-decision of the Retell choice, which is not needed — only the reference is broken, not the decision itself.
- Option 3: violates the append-only invariant and would undermine trust in the ADR system.

## Consequences

- **Good:** ADR-0006 reference is now traceable; workers can follow the chain: ADR-0006 → ADR-0011 → NON_GOALS.md NG-6 + AGENTS.md §7
- **Neutral:** If NON_GOALS.md gains a dedicated NG for mid-sprint platform lock in the future, this ADR should be noted as the historical record of the gap
- **Neutral:** The underlying Retell choice in ADR-0006 is unchanged

## Confirmation

- Any worker reading ADR-0006 and encountering `NG-something equivalent` should follow this ADR (ADR-0011) for the canonical reference
- If a new NON_GOALS.md entry for platform-switch stability is added, update this ADR with the real NG ID (a new superseding ADR is not required — this ADR's body can be referenced as the chain)

## Related

- Erratum for: ADR-0006 (`docs/decisions/0006-retell-default-voice.md`)
- Related: NON_GOALS.md NG-6
- Related: AGENTS.md §7 "Never" list
- Related audit finding: AUDIT-factory-dd92db4f (nightly audit 2026-05-29)

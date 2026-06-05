---
adr: "0011"
title: "Retell AI voice platform — erratum correction (supersedes ADR-0006)"
status: accepted
date: 2026-06-04
deciders: [sebas]
consulted: []
informed: [voice-discovery]
tags: [voice, integration]
supersedes: "0006"
---

# ADR-0011: Retell AI voice platform — erratum correction

## Context and Problem Statement

ADR-0006 (accepted 2026-05-15) contained an unresolved placeholder: "per NON_GOALS.md NG-something equivalent". The 2026-05-30 nightly audit (finding AUDIT-factory-3622c711) appended an `## Erratum` section directly to ADR-0006's body to clarify the placeholder. This violated the append-only rule for accepted ADRs (AGENTS.md §7: "Never edit a `status: accepted` ADR — append-only via new ADR with `supersedes:`"). This ADR supersedes 0006 to establish a clean record.

## Decision

**Retell AI remains the default voice platform** for The Factory's voice interviewer, pending Esme R1 confirmation (now overdue from SPRINT-2026-05-A which ended 2026-05-20).

## Correcting the ADR-0006 placeholder

The phrase "per NON_GOALS.md NG-something equivalent" in ADR-0006's Decision Outcome referenced **SPRINT-2026-05-A §No-gos item 5**: "NOT migrating from Retell if R1 research suggests Vapi is marginally cheaper. Switching mid-sprint = sprint death. Locked for the sprint."

That was a sprint-local no-go for SPRINT-2026-05-A only. It is **not** a permanent entry in `NON_GOALS.md`. The broader principle (avoid platform-switching mid-sprint) is in AGENTS.md §7 "Never" boundaries for mid-sprint architectural pivots.

## Consequences

Same as ADR-0006 (vendor lock-in mitigated by platform-agnostic prompt/tools design). Additionally:

- If Esme R1 research recommends a different platform, open ADR-0012 superseding this one.
- ADR-0006 body is retained as audit history per append-only convention; its status is now `superseded`.

## Related

- Supersedes: ADR-0006
- Related: ROADMAP [M-005] [M-006]
- Related: INTEGRATIONS.md "Retell AI" entry
- Related risks: R-004, R-005

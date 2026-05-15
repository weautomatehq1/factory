---
adr: "0007"
title: "Mermaid + C4 Levels 1-2 only for diagrams"
status: accepted
date: 2026-05-15
deciders: [sebas]
consulted: [claude-orchestrator, architecture-research]
informed: [ifleet, voice-discovery, spec-template]
tags: [architecture, tooling]
supersedes: null
---

# ADR-0007: Mermaid + C4 Levels 1-2 only for diagrams

## Context and Problem Statement

ARCHITECTURE.md and similar docs benefit from diagrams. Multiple options exist: Mermaid (plain-text in markdown), Structurizr (purpose-built C4 DSL), PlantUML (UML-flavored DSL), draw.io (drag-and-drop), image embeds. What's our standard?

## Decision Drivers

- Plain-text diff-ability (agents and humans both review changes)
- Zero toolchain — don't add a build step to the spec repo
- Rendering must work natively on GitHub, Notion, Linear, VS Code, every markdown viewer we use
- AI agents must be able to generate AND modify diagrams as text (no binary editor required)
- Lower-level diagrams (C4 Component, C4 Code) go stale instantly when agents refactor — agents should derive from source instead

## Considered Options

1. **Mermaid + C4 Levels 1-2 only** — plain-text Mermaid; cap diagrams at System Context and Container View
2. **Structurizr DSL** — C4 model purpose-built; needs Structurizr CLI / Lite to render
3. **PlantUML** — broad UML support; needs PlantUML jar to render
4. **All levels (1-4) of C4 in any tool** — comprehensive
5. **draw.io / Lucidchart binary files** — visual editing

## Decision Outcome

Chosen: **Option 1 — Mermaid + C4 Levels 1-2 only.**

C4 Level 1 (System Context) shows the system box with users and external systems. C4 Level 2 (Container View) shows deployable units inside the system. Levels 3 (Component) and 4 (Code) are NOT diagrammed — agents derive structure from source via grep / Read.

### Why not the others?
- **Structurizr DSL:** purpose-built but adds a toolchain (Structurizr CLI or Lite server). For a markdown-only spec repo, this is overhead without proportional gain. Per C4 model's own docs: "notation independent" and "tooling independent."
- **PlantUML:** broad but requires Java + jar to render locally. Not worth the dependency.
- **All 4 C4 levels:** Component and Code diagrams go stale immediately when agents refactor. Stale diagrams are worse than no diagrams — they mislead.
- **draw.io binaries:** not diffable; agents can't generate/modify; review by inspecting commits is impossible.

## Consequences

- **Good:**
  - Zero toolchain — Mermaid renders natively on GitHub
  - Diagrams diff cleanly in PRs
  - Agents generate and modify diagrams as text
  - System Context + Container View capture what humans actually need to onboard
- **Bad:**
  - No native C4 styling (Mermaid doesn't have the canonical C4 shape vocabulary). Acceptable — semantic content > visual fidelity for our scale.
  - Levels 3 and 4 invisible at diagram level. Mitigation: matklad "codemap" section in ARCHITECTURE.md covers what Level 3 would show, derived from source structure.
- **Neutral:**
  - We accept Mermaid's expressiveness ceiling. If we hit it, revisit.

## Confirmation

- Verify: ARCHITECTURE.md renders correctly on GitHub
- Verify: an agent can ADD a new container to the Container View diagram without breaking the file
- Audit: no diagrams in the repo go more than 90 days without being touched after their associated code changes (proxy: are diagrams keeping up with reality?)

## Related

- Related: ARCHITECTURE.md (uses this format)
- Source: matklad ARCHITECTURE.md template (https://matklad.github.io/2021/02/06/ARCHITECTURE.md.html); C4 model (https://c4model.com)

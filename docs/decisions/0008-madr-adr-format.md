---
adr: "0008"
title: "MADR format for ADRs; append-only via filename"
status: accepted
date: 2026-05-15
deciders: [sebas]
consulted: [claude-orchestrator, architecture-research]
informed: [ifleet, voice-discovery, spec-template]
tags: [conventions, docs]
supersedes: null
---

# ADR-0008: MADR format for ADRs; append-only via filename

## Context and Problem Statement

The Factory needs a format for Architectural Decision Records. Options: Michael Nygard's original 2011 format (5 sections), MADR (YAML frontmatter + structured body), Y-Statement (one-sentence), CNCF template. Pick one and lock it.

## Decision Drivers

- Agent-parseable (AI workers need to filter by status, date, tags)
- Append-only enforced structurally, not by prose discipline
- Forces explicit alternatives (so we don't fall in love with the first option)
- Includes "Consequences" so agents downstream know what NOT to break
- Lightweight enough that an ADR can be written in <30 min

## Considered Options

1. **Nygard 2011** — Title, Status, Context, Decision, Consequences
2. **MADR (latest)** — YAML frontmatter (status, date, deciders, consulted, informed, tags, supersedes) + body sections (Context & Problem, Decision Drivers, Considered Options, Decision Outcome, Consequences, Confirmation)
3. **Y-Statement** — One sentence: "In the context of X, facing Y, we decided for Z to achieve W, accepting V"
4. **CNCF ADR template** — similar to MADR with cloud-native flavor

## Decision Outcome

Chosen: **MADR.** Plus filename-based append-only enforcement.

- One ADR per file at `docs/decisions/NNNN-kebab-title.md`
- Sequential numbering, never reused
- YAML frontmatter required (status, date, deciders, tags, supersedes)
- `status: accepted` files are append-only — supersede via new ADR with `supersedes: NNNN`
- Pre-commit hook (planned) blocks edits to accepted ADRs except the one-time `supersedes` flip

### Why not the others?
- **Nygard 2011:** prose-heavy, agent-unfriendly. Filtering by status requires NLP. MADR's YAML frontmatter solves this.
- **Y-Statement:** too dense for agents — no "Consequences" means a worker doesn't know what not to break.
- **CNCF template:** the primary CNCF template URL 404'd during research; the standard is fragmented. MADR is the documented community successor.

## Consequences

- **Good:**
  - YAML frontmatter is agent-parseable (status, date, tags filterable)
  - "Considered Options + Decision Outcome" forces explicit alternatives
  - Append-only enforced by filename, not prose
  - Each file is independent; merge conflicts are nearly impossible
- **Bad:**
  - More overhead per ADR than Y-Statement (5 min savings × 10 ADRs = 50 min total; acceptable)
  - DECISIONS.md must be regenerated from frontmatter (manual until generator wired; automated later)
- **Neutral:**
  - Format may need to be updated when MADR project evolves. We commit to MADR's "latest" — track and update accordingly.

## Confirmation

- Verify: ADRs 0001–0010 are written in this format (initial scaffold)
- Verify: pre-commit hook (planned in IFleet evolution) rejects edits to accepted ADRs
- Verify: DECISIONS.md index parses correctly from `docs/decisions/*.md` frontmatter
- Audit: any ADR where the author was tempted to skip "Considered Options" → was the decision truly self-evident, or did we cut corners?

## Related

- Source: MADR (https://adr.github.io/madr/)
- Related: DECISIONS.md (index); docs/decisions/_template.md
- Self-referential: this ADR is itself in MADR format

---
adr: "0001"
title: "AGENTS.md as primary, CLAUDE.md as @AGENTS.md bridge"
status: accepted
date: 2026-05-15
deciders: [sebas]
consulted: [esme, claude-orchestrator, agent-conventions-research]
informed: [ifleet, voice-discovery, spec-template]
tags: [agents, conventions]
supersedes: null
---

# ADR-0001: AGENTS.md as primary, CLAUDE.md as @AGENTS.md bridge

## Context and Problem Statement

The Factory uses multiple AI agents (Claude Code, OpenAI Codex, Cursor, Aider — Esme also runs Claude Code on Windows). Each agent has historically read its own file: `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, `CONVENTIONS.md`. Duplication means drift. How do we tell every agent the same rules without writing the rules five times?

## Decision Drivers

- Must work for ALL agents we use (Claude Code primary; Codex planned)
- Must avoid duplicating rules across files
- Must survive `/compact` and similar context-management operations
- Must follow conventions emerging in the broader ecosystem (not invent our own)
- Must be easy for a future agent type to adopt without re-engineering our docs

## Considered Options

1. **Parallel files** — maintain `AGENTS.md` and `CLAUDE.md` and `.cursorrules` separately, sync manually
2. **CLAUDE.md primary + symlinks** — write rules in CLAUDE.md, symlink AGENTS.md → CLAUDE.md
3. **AGENTS.md primary + Claude bridge** — write all shared rules in AGENTS.md; CLAUDE.md is a small file that imports it via `@AGENTS.md` (officially documented Anthropic pattern) and adds only Claude-specific behaviors

## Decision Outcome

Chosen: **Option 3 — AGENTS.md primary, CLAUDE.md as `@AGENTS.md` bridge.**

Reasons tied to drivers:
- The `agents.md` convention (https://agents.md) has won as the cross-vendor standard in 2025–2026. Cursor, Codex, Jules, Factory, Amp, Aider, and (via the import bridge) Claude Code all read it.
- Anthropic's docs explicitly recommend the `@AGENTS.md` import: *"If your repository already uses AGENTS.md for other coding agents, create a CLAUDE.md that imports it so both tools read the same instructions without duplicating them."*
- Compaction survival: Anthropic confirms project-root `CLAUDE.md` is re-injected after `/compact`. Nested CLAUDE.md files are not. Keeping AGENTS.md at root (with CLAUDE.md as the root bridge) means load-bearing rules survive long-running sessions.
- Symlinks (Option 2) work on macOS but break on Windows without admin rights — Esme would be blocked.

### Why not the others?
- **Option 1 (parallel files):** documented anti-pattern; rules drift, agents disagree, debugging gets ambiguous.
- **Option 2 (CLAUDE.md primary + symlinks):** Windows admin-rights problem for Esme; symlinks are also brittle in git operations.

## Consequences

- **Good:**
  - One source of truth for agent rules
  - Cross-vendor compatibility from day one
  - Anthropic-officially-supported import pattern (won't break)
  - Esme on Windows works fine
- **Bad:**
  - Slight cognitive overhead for new contributors (two files to know about, even if one is tiny)
  - `@AGENTS.md` import syntax is Claude-specific; other tools just read the AGENTS.md file directly
- **Neutral:**
  - We accept the slight redundancy in having a CLAUDE.md at all — it's the bridge, not duplication

## Confirmation

- Verify: Claude Code session reads AGENTS.md content correctly when only CLAUDE.md is present at root with `@AGENTS.md` first line
- Verify: Codex CLI run in the same repo reads AGENTS.md directly without CLAUDE.md
- Verify: Cursor reads AGENTS.md
- Periodically check: when new agent tools emerge, do they support AGENTS.md? If a tool stops doing so, escalate

## Related

- Related ARCHITECTURE.md section: `#5-architectural-invariants`
- Source: agents.md spec (https://agents.md); Anthropic docs (https://code.claude.com/docs/en/memory)
- Related: AGENTS.md §1, CLAUDE.md (root)

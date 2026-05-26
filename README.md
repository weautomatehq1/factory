# The Factory

> WeAutomateHQ's autonomous client-SaaS factory.

A voice AI interviews a client → produces 17 spec markdown files → autonomous Claude/Codex workers (IFleet) read the spec → build, ship, and self-heal the platform.

**Status:** spec-defining phase. Not yet operational. Initial sprint (SPRINT-2026-05-A) completed 2026-05-21 without full feature delivery; next sprint targeting first operational build.

## What this repo contains

This is the **master coordination repo** for The Factory. It contains:

- **17 spec files** describing The Factory itself (we eat our own dogfood — The Factory is its own first client)
- **Architectural Decision Records** (`docs/decisions/`) — the why behind the picks
- **Operational state** (`.omc/STATUS.md`) — current sprint progress

This repo does **not** contain runnable code. Code lives in the other three Factory repos:

| Repo | Purpose | Status |
|---|---|---|
| `weautomatehq1/IFleet` | Worker fleet — picks issues, ships PRs | Exists (launched 2026-05-19, PR #117) |
| `weautomatehq1/voice-discovery` | Voice interviewer + n8n orchestration | Not yet created |
| `weautomatehq1/spec-template` | GitHub template used to spawn per-client repos | Not yet created |
| `weautomatehq1/<client-name>` | Per-client SaaS — generated from spec-template | Created when client #1 interviews |

## How to use this repo

**As a human (Sebastian or Esme):** read `README.md` (this file) → `ARCHITECTURE.md` → `SPRINT.md` to know what's active.

**As an AI worker (Claude / Codex / Cursor / Aider):** read `AGENTS.md` first. It has the required-reading order, boundaries, commit protocol, and "when stuck" guidance.

## The 17 spec files

| File | Purpose |
|---|---|
| `AGENTS.md` | Rulebook for AI workers (cross-vendor agents.md convention) |
| `CLAUDE.md` | Claude-specific bridge (`@AGENTS.md` import + Claude rules) |
| `ROADMAP.md` | Phased milestones with `[M-NNN]` IDs |
| `ARCHITECTURE.md` | System shape (Mermaid C4 Levels 1–2) + invariants |
| `SPRINT.md` | Current active work |
| `DECISIONS.md` | Auto-generated index of `docs/decisions/*.md` ADRs |
| `UBIQUITOUS_LANGUAGE.md` | Domain glossary (client's word wins) |
| `FILE_STRUCTURE.md` | Directory conventions |
| `INTAKE.md` | Verbatim client interview (skeleton until client #1) |
| `NON_GOALS.md` | Explicit out-of-scope (Non-Goals = "not this release", No-Gos = "never") |
| `STAKEHOLDERS.md` | DACI decision rights |
| `RISKS.md` | FMEA-scored risk register |
| `KPI.md` | Success metrics (skeleton until launch) |
| `RUNBOOK.md` | Operational playbook (skeleton until first incident) |
| `ENV.md` | Env var registry (names only, never values) |
| `INTEGRATIONS.md` | Third-party services + auth model |
| `SECURITY.md` | Threat model + protected paths + data classification |
| `CHANGELOG.md` | Release log (Keep a Changelog + SemVer) |

## Glossary (quick)

- **The Factory** — this entire system (interview → spec → build → self-heal)
- **IFleet** — the autonomous Claude/Codex worker pool
- **Voice-discovery** — the Retell-AI-powered interviewer
- **Spec-template** — the GitHub template each client project starts from
- **Decomposer** — the IFleet component that reads merged spec → emits GitHub issues
- **Self-healing** — three-tier system (build / deploy / runtime) that auto-fixes failures within bounded autonomy

Full glossary: `UBIQUITOUS_LANGUAGE.md`.

## Conventions

- **Conventional Commits** (`type(scope): description`)
- **Plain-English recap** after every technical explanation to Sebastian (rule in `~/.claude/CLAUDE.md`)
- **Append-only ADRs** — supersede via new file; never edit accepted ones
- **One responsibility per file** — `~/.claude/CLAUDE.md` "Beast Mode" standards apply

## Team

- **Sebastian Puig** — founder, Driver, primary Approver
- **Esme** — collaborator, builder, perfectionist (on Windows — see briefs for shell-flavor adjustments)

## Related

- Sebastian's global rules: `~/.claude/CLAUDE.md`
- Auto-memory index: `~/.claude/projects/-Users-Seb/memory/MEMORY.md`
- IFleet status (latest): check MEMORY.md index for the most recent `project_status_*.md` entry (do not rely on a hardcoded filename — it is updated per session)

---

**Last updated:** 2026-05-26

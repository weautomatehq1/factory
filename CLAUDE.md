@AGENTS.md

# CLAUDE.md — Claude Code specific overrides

The `@AGENTS.md` import above pulls in all shared agent rules. This file adds only Claude-Code-specific behaviors that Codex / Cursor / Aider don't share.

## Required reading at session start

1. `.omc/STATUS.md` — what the last session left behind
2. The 18 spec files in the order defined in `AGENTS.md` §2 (skim, deep-read only the ones relevant to your task)
3. The nearest `AGENTS.md` to the files you're about to edit (cascading rule)

## Default mode

- Use **plan mode** (`Shift+Tab`) for any task touching 2+ files, an unfamiliar area, or a `status: accepted` artifact.
- Skip plan mode for typos, single-section edits, or rename-only changes.

## Verification (mandatory before declaring done)

- For markdown repos: confirm GitHub renders correctly + cross-references resolve.
- For code repos: run the repo's `pnpm lint && pnpm typecheck && pnpm test`. Paste the result in your final message.
- For UI changes: screenshot via Chrome extension, compare to design in spec.
- **Never claim "done" without verifying.**

## Subagent delegation

- **Multi-file refactor** → `executor` subagent (model: `sonnet`; `opus` for security/billing/auth).
- **Research / exploration** → research subagent in read-only mode; main thread applies the edits.
- **Always run a separate-context reviewer pass before commit.** Never self-approve.

## Context hygiene

- `/clear` between unrelated tasks.
- `/compact` only when the current task is still active; never compact between tasks.
- For exploratory questions, use `/btw` (sidebar) so answers don't pollute the main thread.

## Continuous execution (default, not a keyword mode)

Per `~/.claude/CLAUDE.md` "Continuous execution" rule: after every completed task, check the queue, then the spec (`SPRINT.md` → `ROADMAP.md` → `ARCHITECTURE.md`). Generate next tasks if queue is empty. Stop only on: real blocker, milestone hit, cost ceiling, KPI regression, 3 consecutive failures, or `SECURITY.md` protected-path touch.

## Hooks (configured in `.claude/settings.json`)

- `PreToolUse:Bash` — blocks `git push --force` to `main`
- `PreCommit` — rejects secret patterns in diff
- `Stop` — reminds to update `.omc/STATUS.md` if any files changed

---

**Last updated:** 2026-05-15

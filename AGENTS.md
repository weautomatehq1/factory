# AGENTS.md

> ⚠️ **WARNING — hard-enforcement controls newly wired (2026-05-21).** §13 below lists "hard-enforced rules" (pre-commit secret rejection, accepted-ADR edit blocker, force-push blocker). Until 2026-05-21 these were aspirational — the hook files did not exist. The first PR (`fix(audit): close AUDIT-factory-b6ff2b68`) creates `.husky/pre-commit`, `.github/CODEOWNERS`, `.github/workflows/protected-paths.yml`, and `.claude/settings.json`. Workers and humans MUST run `git config core.hooksPath .husky` after first clone for the secret-scan hook to fire locally. The first Protected-Path PR is the live verification of the gate.

> This file is the primary rulebook for every AI worker that touches this repo. It follows the cross-vendor agents.md convention (https://agents.md). Claude Code reads it via the `@AGENTS.md` import in `CLAUDE.md`.

## 1. Project overview

**The Factory** is WeAutomateHQ's autonomous client-SaaS factory. A voice AI interviews a client → produces 17 spec files in this format → autonomous Claude/Codex workers (IFleet) read the spec → build, ship, and self-heal the platform.

Two repos sit beside this one:
- `github.com/weautomatehq1/IFleet` — the worker fleet that executes specs
- `github.com/weautomatehq1/voice-discovery` — the voice interviewer (to be created)

Plus one per client: `github.com/weautomatehq1/<client-name>` — generated from this template at interview-end.

**Audience:** this file is read by AI workers (Claude, Codex, Cursor, Aider, Amp). Humans use `README.md`.

## 2. Required reading order

Before doing any non-trivial work in a client repo generated from this template, read these in order:

1. `INTAKE.md` — verbatim client interview (the source of truth — never invent)
2. `NON_GOALS.md` — explicit out-of-scope (kills gold-plating)
3. `UBIQUITOUS_LANGUAGE.md` — domain vocab; client's word always wins
4. `ARCHITECTURE.md` — system shape + invariants
5. `ROADMAP.md` — phased milestones with `[M-NNN]` IDs
6. `SPRINT.md` — active sprint (your work queue)
7. `FILE_STRUCTURE.md` — where files go
8. `SECURITY.md` — protected paths (auto-merge gates)
9. `STAKEHOLDERS.md` — who decides what
10. `INTEGRATIONS.md` — third-party services + auth model
11. `ENV.md` — env var registry
12. `KPI.md` — success metrics (when populated)
13. `RISKS.md` — known risks + mitigations
14. `RUNBOOK.md` — operational playbook (when populated)
15. `DECISIONS.md` → `docs/decisions/*.md` — why we picked what we picked
16. `CHANGELOG.md` — release history
17. `AGENTS.md` — this file
18. `CLAUDE.md` — Claude-specific bridge

**If a spec file contradicts code, the spec wins.** Open an issue; do not silently drift.

## 3. Repo structure

```
factory/
├── AGENTS.md                  # this file
├── CLAUDE.md                  # Claude-specific bridge (imports AGENTS.md)
├── README.md                  # human orientation
├── CHANGELOG.md
├── ROADMAP.md
├── ARCHITECTURE.md
├── SPRINT.md
├── FILE_STRUCTURE.md
├── UBIQUITOUS_LANGUAGE.md
├── INTAKE.md                  # status: skeleton until client #1
├── NON_GOALS.md
├── STAKEHOLDERS.md
├── RISKS.md
├── KPI.md                     # status: skeleton until launch
├── RUNBOOK.md                 # status: skeleton until first incident
├── ENV.md
├── INTEGRATIONS.md
├── SECURITY.md
├── DECISIONS.md               # auto-generated index
├── docs/decisions/            # one ADR per file (NNNN-*.md)
│   ├── _template.md
│   └── 0001-*.md … 0010-*.md
└── .omc/
    ├── STATUS.md              # Done / In flight / Up next
    └── costs.json             # usage log
```

### Nested AGENTS.md expansion clause

This file is the only `AGENTS.md` for now. **Nested `AGENTS.md` files are added when a subfolder earns specific rules** (e.g., `apps/api/auth/AGENTS.md` with stricter "ask first" boundaries). Do not create empty nested files. When a worker proposes adding one, it must justify why root-level rules are insufficient.

When nested files exist, the nearest one wins (closest to the edited file). The agents.md spec defines this cascading behavior.

## 4. Setup

```bash
git clone git@github.com:weautomatehq1/factory.git ~/dev/coordination/factory
cd ~/dev/coordination/factory
# no install step — this repo is markdown only
```

The Factory itself runs no code. Code lives in `IFleet`, `voice-discovery`, and per-client repos.

## 5. Commands you can run unattended

This repo is markdown only — no build, no tests. Worker actions in this repo:

| Action | Tool | When |
|---|---|---|
| Add an ADR | `Write` new file in `docs/decisions/NNNN-*.md` | When making a non-trivial decision |
| Update `DECISIONS.md` index | regen from frontmatter | After adding an ADR |
| Update `CHANGELOG.md` `[Unreleased]` | `Edit` | On every spec change |
| Update `.omc/STATUS.md` | `Edit` | End of each work session |

For action rules in IFleet / voice-discovery / client repos, read those repos' own `AGENTS.md`.

## 6. Code standards (delta from defaults)

This repo has no code. See per-repo `AGENTS.md` for IFleet / voice-discovery / client conventions. The standards there inherit from Sebastian's global `~/.claude/CLAUDE.md` (Beast Mode) which is the source of truth for TypeScript / React / API / security / git hygiene.

## 7. Boundaries

Steal Cloudflare's `Always / Ask first / Never` pattern. Every rule has a one-line **Rationale** so workers can apply the spirit, not just the letter.

### Always
- **Read the relevant spec files before editing.** Rationale: spec is the contract; drift creates rework.
- **Write past-tense events in `UBIQUITOUS_LANGUAGE.md`** (e.g., `MatterOpened`, not `OpenMatter`). Rationale: Event Storming convention — these become event names in code later.
- **Cite source for every claim in INTAKE.md** (timestamp + speaker). Rationale: INTAKE is raw evidence; synthesized claims belong elsewhere.
- **Use the client's words in `UBIQUITOUS_LANGUAGE.md`** verbatim. Rationale: ambiguity here cascades into wrong schemas, wrong UI labels, wrong tickets.
- **Append-only on `docs/decisions/`** — supersede via new ADR, never edit accepted ones. Rationale: decision history must be auditable.
- **Update `.omc/STATUS.md` at end of every session.** Rationale: next agent reads this first.
- **Run the plain-English recap rule** when explaining anything technical to Seb. Rationale: defined in `~/.claude/CLAUDE.md`.

### Ask first (open an issue, don't just do it)
- **Modifying any file with `status: accepted` in frontmatter** (ADRs, accepted SPRINT, locked ARCHITECTURE invariants). Rationale: these are commitments; changing them needs the original Approver's sign-off.
- **Adding to `NON_GOALS.md`'s "No-Gos" list.** Rationale: a "never" is forever; it must be deliberate.
- **Adding a new integration** to `INTEGRATIONS.md`. Rationale: each adds attack surface; security review required.
- **Touching `SECURITY.md` `Protected Paths` list.** Rationale: this is the auto-merge gate; loosening it weakens the whole safety model.
- **Changing the file format of any of the 17 spec files** (frontmatter schema, section names). Rationale: the decomposer parses these; format changes break it.

### Never
- **Commit secrets, `.env`, or API keys.** Rationale: public repo; one leak = full rotation.
- **Fabricate INTAKE content.** If we don't have a client interview yet, status stays `skeleton`. Rationale: agents downstream cite INTAKE; fake intake = fake spec = fake product.
- **Edit a `status: accepted` ADR.** Rationale: append-only via new ADR with `supersedes:`.
- **Delete files in `memory/` (any repo).** Rationale: other agents and humans wrote them; you don't know what's load-bearing.
- **Use `Get-Content` / `Set-Content` in PowerShell.** Rationale: corrupts encoding (Esme is on Windows; this matters cross-team).
- **Force-push to `main`.** Rationale: rewrites history; loses other agents' work.
- **Skip pre-commit hooks (`--no-verify`).** Rationale: hooks are the only hard enforcement (per Anthropic docs — AGENTS.md/CLAUDE.md are advisory).
- **Generate `.yaml` siblings of spec files unless an agent has failed to parse the `.md`.** Rationale: ADR-0001 — premature sync layer = drift risk.

## 8. Testing

This repo has no tests (markdown only). For client repos, see NON_GOALS.md NG-7 (smoke-test-first rationale) and Sebastian's global `~/.claude/CLAUDE.md`. Note: ADR-0008 covers ADR format, not testing philosophy.

For changes here, the verification is:
- Cross-references resolve (linked sections exist)
- Frontmatter is valid YAML
- Markdown renders correctly on GitHub
- Spell-check + grammar on changed prose

## 9. Commit & PR protocol

- **Branch from `main`:** `feat/<short>`, `fix/<short>`, `chore/<short>`, `docs/<short>`, `adr/<short>`.
- **Conventional Commits:** `<type>(<scope>): <imperative description>`.
  - Example: `docs(non-goals): add explicit no-go for native mobile app`
  - Example: `chore(adr): add ADR-NNNN for n8n vs Inngest`
- **One logical change per commit.** Don't combine an ADR with a roadmap edit.
- **PR title:** `[<area>] <imperative title>`. Example: `[spec] Add Q3 milestones to ROADMAP`.
- **PR body MUST include:**
  - **Summary** (1–3 bullets)
  - **Spec reference** (which spec file this touches or implements)
  - **Test plan** (for code repos: bulleted checklist · for this repo: "rendered preview reviewed")
  - **Out of scope** (what was deliberately left out)
- **Never `--no-verify`. Never force-push to a shared branch. Never self-merge.** A human or designated reviewer agent merges.
- **The agent does NOT self-merge.** Even on green CI.

## 10. Memory & context

- **`memory/` (in IFleet / voice-discovery / client repos)** — shared agent memory. Read at session start. Append at session end if you discovered something the next agent needs to know.
- **`memory/handoff.md`** — what the previous agent left for the next. Required reading.
- **Sebastian's auto-memory** lives at `~/.claude/projects/-Users-Seb/memory/MEMORY.md`. Indexed; per-agent. Cross-reference with `[[name]]` syntax.
- **Compaction survival:** the root `AGENTS.md` and `CLAUDE.md` are re-injected after `/compact`. Load-bearing rules live here, NOT in nested files (per Anthropic docs).

## 11. When stuck

If you cannot find the answer in `INTAKE.md`, `ARCHITECTURE.md`, this file, or `memory/`:

1. **Stop.** Do not invent.
2. **Check** `docs/decisions/` — there may be an ADR that answers it.
3. **Open a GitHub issue** labeled `clarification-needed` with: what you tried, what's ambiguous, your proposed conservative default.
4. **If running autonomously (overnight ralph mode):** pick the most conservative interpretation, log it in a new ADR (`status: proposed`), proceed. Sebastian reviews in the morning.
5. **If the gap is in INTAKE.md:** flag it as an `open_loop` in INTAKE itself for the next discovery call.

## 12. Glossary

Domain terms live in `UBIQUITOUS_LANGUAGE.md`. Do not duplicate definitions here. If you find yourself wanting to define a term in code or commit, check UBIQUITOUS first; add an entry if missing.

## 13. Hard enforcement vs advisory rules

This file is **advisory**. Per Anthropic's docs, AGENTS.md and CLAUDE.md shape agent behavior but are not enforced — the agent can choose to ignore them.

**For absolutely-never rules**, use hooks in `.claude/settings.json` (Claude Code) or sandbox config (Codex). Current hard-enforced rules:

- Pre-commit hook: rejects commits with secret patterns (`sk_`, `xoxb-`, JWT `eyJ`)
- Pre-commit hook: rejects edits to `docs/decisions/*.md` with `status: accepted` (allows `proposed → accepted/rejected/superseded` transitions only)
- Pre-push hook: blocks `push --force` to `main`

If a rule in this file MUST be unbreakable, promote it to a hook. Note the promotion here.

---

**Last updated:** 2026-05-26
**Last verified:** 2026-05-26 — nightly audit (read-only review; no content changes)

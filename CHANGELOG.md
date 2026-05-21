# Changelog

All notable changes to The Factory will be documented in this file.

The format is based on [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed
- **2026-05-21 — Sprint reconciliation** (closes audit finding `AUDIT-factory-b5bd1864`). SPRINT-2026-05-A `ends: 2026-05-20` arrived but `git log --since=2026-05-15` showed zero feature commits in this repo since the initial scaffold (`d49b1a4`). Docs that implied progress were rolled back to git reality without fabricating completion: SPRINT.md `status: active → incomplete`, all M-NNN tasks marked `⏭️ Deferred` or `❌ Not started`, ROADMAP.md M-001 `in-progress → deferred`, `.omc/STATUS.md` rewritten with honest snapshot. Sibling-repo work (IFleet, voice-discovery) NOT backfilled here — that requires a separate PR with explicit cross-repo evidence. SPRINT-2026-05-B re-plan pending.

### Added
- Initial scaffold of The Factory master coordination repo at `~/dev/coordination/factory/`
- `AGENTS.md` — cross-vendor AI worker rulebook (agents.md convention)
- `CLAUDE.md` — Claude-specific bridge importing AGENTS.md
- `README.md` — human orientation
- `ROADMAP.md` — 15 milestones for 5-day pipeline build sprint
- `ARCHITECTURE.md` — system shape (Mermaid C4 Levels 1–2) + 10 architectural invariants
- `SPRINT.md` — SPRINT-2026-05-A active sprint definition (Shape Up adapted)
- `FILE_STRUCTURE.md` — directory conventions for The Factory + template for client repos
- `UBIQUITOUS_LANGUAGE.md` — 22 domain terms + 24 events (past-tense, Event Storming convention)
- `INTAKE.md` — skeleton (populated when first real client interview captured)
- `NON_GOALS.md` — 8 Non-Goals + 10 No-Gos
- `STAKEHOLDERS.md` — DACI decision rights + decision log (18 historical decisions)
- `RISKS.md` — FMEA-scored risk register, 15 open risks
- `KPI.md` — skeleton (populated when first client production data captured)
- `RUNBOOK.md` — skeleton (populated per Google SRE "rule of two")
- `ENV.md` — env var registry (names only, never values), 24 vars across 7 categories
- `INTEGRATIONS.md` — 14 third-party services + 4 internal tools registered
- `SECURITY.md` — STRIDE threat model + OWASP Top 10 mapping + ASVS L2 default + Protected Paths glob (the auto-merge gate)
- `DECISIONS.md` — auto-generated ADR index (placeholder until generator wired)
- `docs/decisions/_template.md` — MADR template for future ADRs
- `docs/decisions/0001-agents-md-as-primary.md` through `0010-issues-from-four-upstream-sources.md` — 10 retroactive ADRs capturing decisions made 2026-05-12 → 2026-05-15
- `.omc/STATUS.md` — Done / In flight / Up next operational state
- `.omc/costs.json` — empty usage log

### Decisions
- Use `AGENTS.md` as primary AI rulebook, `CLAUDE.md` as `@AGENTS.md` import bridge (ADR-0001)
- Four-repo structure: Factory + IFleet + voice-discovery + spec-template + per-client (ADR-0002)
- GitHub Issues remain the unit of work for IFleet, fed by 4 upstream automated sources + humans (ADR-0003, ADR-0010)
- Build the full pipeline before client #1 — no manual shortcuts (ADR-0004)
- Tier 3 runtime self-healing deferred until 2 weeks of production data (ADR-0005)
- Retell AI as voice platform default, pending Esme R1 confirmation (ADR-0006)
- Mermaid + C4 Levels 1–2 only for diagrams; no Structurizr / PlantUML toolchain (ADR-0007)
- MADR format for ADRs, append-only via filename discipline (ADR-0008)
- Four-tier data classification (Public / Internal / Confidential / Restricted) with NIST C/I/A mapping (ADR-0009)

### Security
- Pre-commit hook design: rejects commits matching secret patterns (`sk_`, `xoxb-`, JWT `eyJ`, AWS keys, GitHub PATs)
- Protected Paths defined in `SECURITY.md` — auto-merge gates honor this list absolutely

---

<!--
Categories (only these six):
### Added       — new features
### Changed     — changes to existing functionality
### Deprecated  — soon-to-be removed
### Removed     — now-removed features
### Fixed       — bug fixes
### Security    — vulnerability-related changes (always present after security event)

SemVer bumping rules:
- Breaking change in Changed or Removed → MAJOR
- Entry in Added → MINOR
- Only Fixed / Security → PATCH

On release:
- Promote [Unreleased] to [X.Y.Z] — YYYY-MM-DD
- Reset [Unreleased] empty
- Tag release in git
-->

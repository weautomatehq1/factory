# Changelog

All notable changes to The Factory will be documented in this file.

The format is based on [Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Changed
- **2026-06-10 — Nightly audit fixes** (5 findings fixed — 3 IMPORTANT, 2 COSMETIC). KPI.md: KR3 deadline reached today (2026-06-10) confirmed MISSED; "3 days remain" corrected to "deadline reached"; frontmatter last_updated 2026-05-15→2026-06-10. RUNBOOK.md: frontmatter last_updated 2026-05-26→2026-05-30 (matching footer). SECURITY.md: frontmatter last_updated 2026-05-21→2026-05-26 (matching footer, pre-existing mismatch). ROADMAP.md: Esme R1 ACTION REQUIRED updated from "10 days elapsed" to "21 days elapsed". STATUS.md: In flight updated to day 20 overdue, Done section updated with today's session.
- **2026-06-07 — Nightly audit fixes** (11 findings fixed — 3 CRITICAL, 5 IMPORTANT, 3 COSMETIC). KPI.md: KR2 marked MISSED (11 days past 2026-05-27 deadline; confidence 2→0), KR3 marked MISSED (KR2 prerequisite failed; confidence 2→0), OKR banner updated — all four Q2 OKRs now confirmed MISSED. STATUS.md: stale 2026-06-04 in-flight entry resolved; 2026-06-07 audit session noted. ROADMAP.md: ADR-0006 reference in Q3 updated to ADR-0011, frontmatter timestamps updated, footer updated. INTEGRATIONS.md frontmatter: related_adrs [ADR-0006] → [ADR-0011]. NON_GOALS.md frontmatter: related_adrs [ADR-0004] removed (no formal ADR covers this scope decision). RISKS.md: R-006 "Phase B decision" jargon replaced with traceable reference; R-013 M-026 replaced with "not yet scoped — planned post-[M-024]". CHANGELOG.md: ADR-0011 Added entry added; ENV.md and INTEGRATIONS.md counts corrected.
- **docs/decisions/0011-retell-voice-platform-erratum.md** — ADR-0011 created (2026-06-04) to supersede ADR-0006 per append-only invariant; Retell remains default per ADR-0011.
- **ENV.md expanded**: 9 categories (was 7), 31+ vars (was 24) — Stripe and Documenso sections added in 2026-05-26 audit.
- **INTEGRATIONS.md**: 13 third-party services (was claimed 14 in scaffold entry below; correct count is 13).
- **2026-06-04 — Nightly audit fixes** (15 findings fixed — 8 IMPORTANT, 7 COSMETIC). ADR-0006 erratum-in-body violation resolved: ADR-0011 created as proper superseder, ADR-0006 status flipped to `superseded` — closes AUDIT-factory-20260604-01. ROADMAP.md: reconciliation banner added to "Next" section (window expired today), M-015 depends-on fixed from unparser range to explicit array — closes AUDIT-factory-20260604-02 and -07. KPI.md: KR4 marked MISSED (deadline 2026-06-04; M-018 not built), confidence updated 6→0 — closes AUDIT-factory-20260604-03. AGENTS.md: §8 NG-7 description corrected from "smoke-test-first rationale" to accurate "intentional test-debt trade-off", §2 CLAUDE.md clarified as bridge, not a spec file — closes AUDIT-factory-20260604-04 and -09. SECURITY.md: killswitches M-019 reference corrected (M-019 is cost dashboard, not Discord bridge) — closes AUDIT-factory-20260604-06. UBIQUITOUS_LANGUAGE.md: Protected Path source corrected from ADR-0001 to ARCHITECTURE.md §5 + SECURITY.md, Gap entry clarified with build-sequence vs logical-ID note, Sprint source replaced with accurate prose — closes AUDIT-factory-20260604-05, -11, and -14. DECISIONS.md: ADR-0002 title clarified (4 base repos + N per-client), ADR-0011 added to index — closes AUDIT-factory-20260604-10. CHANGELOG.md: updated UBIQUITOUS_LANGUAGE.md term/event counts — closes AUDIT-factory-20260604-08. INTEGRATIONS.md: Phase B jargon replaced with traceable reference — closes AUDIT-factory-20260604-12. ARCHITECTURE.md: ADR-0005 and ADR-0009 added to related_adrs — closes AUDIT-factory-20260604-13. NON_GOALS.md: NoGo-1 incorrect ADR-0004 citation replaced with accurate source — closes AUDIT-factory-20260604-15.
- **UBIQUITOUS_LANGUAGE.md now contains 27 domain terms + 26 events** (5 terms and 2 events added since initial scaffold on 2026-05-15; initial scaffold entry below still says 22 + 24).
- **2026-05-26 — Nightly audit fixes** (9 findings fixed, 5 remain open). ENV.md: added Stripe (STRIPE_SECRET_KEY, STRIPE_PUBLISHABLE_KEY, STRIPE_WEBHOOK_SECRET) and Documenso (DOCUMENSO_API_KEY) var rows + rotation calendar entries — closes AUDIT-factory-9ea4af13. ROADMAP.md: replaced incomplete Not Doing section with pointer to NON_GOALS.md — closes AUDIT-factory-4e615356 and AUDIT-factory-7217b91c. README.md: fixed stale IFleet memory file reference and updated IFleet status to launched — closes AUDIT-factory-6f96e44a. SECURITY.md: removed invalid (TBD) suffixes from security.txt Encryption and Policy lines per RFC 9116 — closes AUDIT-factory-62468376. costs.json: corrected cross-repo IFleet reference — closes AUDIT-factory-95226593. CODEOWNERS + workflow: added missing `**/auth/**` glob alongside existing `**/(auth)/**` — closes AUDIT-factory-e1a7c3f2. KPI.md: marked KR1 as missed, flagged KR2 at-risk — closes AUDIT-factory-a3b19d77. ROADMAP.md footer date synced with frontmatter — closes AUDIT-factory-8c4e2b56. SECURITY.md footer date synced — closes AUDIT-factory-7f3a9c12. Bulk last_updated stamps refreshed to 2026-05-26 across 13 spec files — closes AUDIT-factory-d567bb1d. STATUS.md updated with current in-flight state. AUDIT-factory-308e0445 (CHANGELOG ADR-0010) confirmed already fixed in previous audit, status updated.
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

---
file: RISKS.md
status: accepted
last_updated: 2026-05-26
owner: sebas
scoring: FMEA — Severity (1-10) × Occurrence (1-10) × Detection (1-10) = RPN
review_threshold: RPN >= 100
mitigate_threshold: RPN >= 200
escalate_rule: "S >= 9 always escalates regardless of RPN (catches rank-reversal weakness of ordinal multiplication)"
---

# Risk Register

> FMEA-scored. Agents may PROPOSE new rows but not change S/O/D scores without human review (HITL). Every postmortem creates or updates a row. Quarterly review re-scores every open row.

## Scoring guides

| Severity (S) | Meaning |
|---|---|
| 1 | No impact |
| 3 | Annoying, fully recoverable |
| 5 | Material delay or rework |
| 7 | Customer-visible degradation |
| 9 | Data loss, security breach, contract violation |
| 10 | Catastrophic (existential risk to project / company) |

| Occurrence (O) | Meaning |
|---|---|
| 1 | Extremely unlikely (theoretical) |
| 3 | Possible but no precedent in our context |
| 5 | Has happened once or twice in similar projects |
| 7 | Recurring pattern in our tools / stack |
| 9 | Near certain in next quarter |
| 10 | Already happening |

| Detection (D) | Meaning |
|---|---|
| 1 | Instant alarm, automated |
| 3 | Caught in pre-commit / CI |
| 5 | Caught in code review or smoke test |
| 7 | Caught in staging |
| 9 | Caught in production by ops |
| 10 | Undetectable until customer reports |

**Status values:** open | mitigating | mitigated | accepted | transferred | closed

---

## Current risks (open + mitigating)

| ID | Risk | Mode of failure | Effect | S | O | D | RPN | Mitigation | Owner | Status | Last reviewed |
|---|---|---|---|---|---|---|---|---|---|---|---|
| R-001 | Voice intake captures wrong spec | LLM hallucinates feature; client says X, transcript stores Y | Wrong product built; weeks of rework | 9 | 4 | 6 | 216 | Verbatim transcript is canonical; synthesis cites timestamps; operator reviews coverage heatmap live; spec PR is human-gated | sebas | mitigating | 2026-05-15 |
| R-002 | Worker auto-merges breaking change to Protected Path | Auto-merge gate fails or path-glob is incomplete | Auth/billing breakage in production | 10 | 3 | 4 | 120 | Protected Paths enforced by CODEOWNERS + dedicated GitHub Action; Tier 3 default OFF; quarterly glob audit (SECURITY.md) | sebas | mitigating | 2026-05-15 |
| R-003 | Secret committed to repo | Pre-commit hook misses a pattern; worker writes a value into a markdown table | Credential leak (public repo); full rotation | 10 | 2 | 7 | 140 | Pre-commit hook enforces secret-pattern regex; ENV.md says "names only"; quarterly secret-pattern audit | sebas | mitigating | 2026-05-15 |
| R-004 | Retell tool-calling state loss mid-call | Interrupt during tool call; long-call (60+min) memory exhaustion | Interview drops, coverage incomplete, client annoyed | 7 | 5 | 5 | 175 | Coverage checkpoint every 5 min; `escalate_to_human` tool exists; operator dashboard "Interrupt" button; D2 R1 research informs platform pick | sebas | mitigating | 2026-05-15 |
| R-005 | n8n + Retell webhook auth failure on first integration | Wrong HMAC implementation; webhook drops calls | Interviews silently fail; client thinks system is broken | 8 | 6 | 6 | 288 | Use n8n built-in webhook auth (not custom HMAC); ship synthetic webhook test before going live; alert if no webhook received within 60s of call ending | sebas | mitigating | 2026-05-15 |
| R-006 | IFleet worker rate-limit on Claude API mid-sprint | Opus 5-hour cap hit on long brief; silent code=1 | Sprint stalls; debugging time wasted | 5 | 7 | 4 | 140 | Phase B decision: sonnet default + opus only on complexity:high; rate-limit detection downshifts; per-project daily rate cap | esme | mitigating | 2026-05-15 |
| R-007 | Spec synthesizer first generation is bad | Prompt-tuning takes more iterations than budgeted | Day 4 slips into Day 5 | 5 | 8 | 3 | 120 | Plan 3-5 iterations in appetite; dry-run on Day 5 catches it; manual spec fix-up is acceptable for client #1 | sebas | mitigating | 2026-05-15 |
| R-008 | Self-heal Tier 2 rollback fails to roll back | Vercel rollback API breaks; manual rollback needed | Bad deploy stays live; customer-visible | 8 | 4 | 6 | 192 | Intentionally break something on Day 5 to test rollback; alert if rollback API returns non-200 within 60s | sebas | mitigating | 2026-05-15 |
| R-009 | Multi-repo coordination breaks (Factory + IFleet + voice-discovery diverge) | Spec format change in Factory not reflected in decomposer | Decomposer fails to parse, no issues created | 7 | 5 | 5 | 175 | Format version in frontmatter; decomposer asserts version; cross-repo CI runs synthesizer dry-run on every Factory change | esme | mitigating | 2026-05-15 |
| R-010 | KPI regression goes undetected | KPI.md remains skeleton; self-heal has nothing to anchor on | Self-heal silently misbehaves post-launch | 7 | 7 | 8 | 392 | Sequenced gate: (1) KPI targets and alert_threshold values MUST be defined before M-016 (launch gate — targets without data are sufficient); (2) KPI.md fully populated with 7 days of production data before M-017 (Tier 3 self-healing go-no-go). Tier 3 defaults OFF until gate (2) clears. Skeleton status reviewed weekly. | sebas | mitigating | 2026-05-23 |
| R-011 | Esme on Windows hits Mac-only commands | Brief contains `pnpm` or zsh-specific syntax not portable | Esme's work delayed; cross-team friction | 4 | 6 | 5 | 120 | Briefs mark commands `[WINDOWS]` / `[MAC]`; never use `Get-Content`/`Set-Content`; cross-platform tooling preferred (pnpm works on Windows) | sebas | mitigating | 2026-05-15 |
| R-012 | Sebastian unavailable during a sprint blocker | Sebastian sick, traveling, family event | Esme blocked on every Protected Path decision | 6 | 5 | 9 | 270 | STAKEHOLDERS.md "Escalation paths" defines fallback; Esme can proceed on non-Protected work; new ADRs proposed in absence | sebas | mitigating | 2026-05-15 |
| R-013 | Client interview reveals scope wildly different from expected | Client wants something The Factory can't build (mobile app, ML pipeline) | Wasted interview; awkward conversation; NoGo-2 invoked | 6 | 4 | 3 | 72 | Pre-call screening questionnaire (planned M-026); INTAKE flags NoGo categories early; Layer 2 redirects out-of-scope topics | sebas | open | 2026-05-15 |
| R-014 | Decomposer creates conflicting / duplicate issues | Parser bug; ambiguous spec section emits same issue twice | Workers do duplicate work; cost waste | 5 | 5 | 6 | 150 | Decomposer dedupes by spec anchor reference; CI on decomposer runs canonical sample; M-002 acceptance criteria includes dedup test | esme | mitigating | 2026-05-15 |
| R-015 | INTAKE.md fabrication temptation | Operator wants to "start somewhere" before real client | Downstream cites fake quotes; worse than no INTAKE | 9 | 3 | 7 | 189 | NoGo-7 makes this explicit; AGENTS.md §7 "Never" list includes fabrication; skeleton status prevents accidental reading-as-truth | sebas | mitigating | 2026-05-15 |

---

## Closed / mitigated risks (for institutional memory)

| ID | Risk | Outcome | Date closed |
|---|---|---|---|
| R-CL-001 | Damian working on parallel real estate project causes context drift | Damian no longer on team | 2026-05-14 |

---

## Risks we accept (won't mitigate further)

- **Cost overrun on a single overnight ralph run:** accepted — that's why `OVERNIGHT_MODE=on` exists. Tradeoff documented in `~/.claude/CLAUDE.md`.
- **Imperfect first-client deliverable:** accepted — client #1 is also our dogfood; some friction is the cost of learning the system.
- **GitHub public-repo nature of IFleet:** accepted (over $4/mo Team upgrade). Secrets stay out via `.gitignore`. See `[[ifleet-repo]]`.

---

## Review schedule

- Weekly (during sprint): re-score open risks where status changed
- Per postmortem: create or update a row
- Quarterly: full re-score; promote `mitigating` → `mitigated` where evidence supports it
- Per ADR creation: check if the decision introduces new risks; add rows

---

**Last updated:** 2026-05-26
**Last verified:** 2026-05-26 — nightly audit (read-only review; no content changes)

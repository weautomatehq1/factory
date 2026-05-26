---
file: STAKEHOLDERS.md
status: accepted
last_updated: 2026-05-26
owner: sebas
---

# Stakeholders

> DACI decision-rights framework (lightest-weight option for a 2-person agency). One named Approver per decision class. Decision log captures the trail.

## Roles

### Approver (exactly one human, per decision class)

| Decision class | Approver | Default response SLA | Out-of-office plan |
|---|---|---|---|
| Scope changes to ROADMAP | Sebastian Puig | 24h | Esme acts as proxy if Sebastian unreachable > 48h, MUST flag in PR description |
| Architecture / ADR | Sebastian Puig | 48h | Same as above |
| Security / Protected Paths | Sebastian Puig | 24h | Never delegated — wait for Sebastian even if blocked |
| Budget / cost-ceiling lifts | Sebastian Puig | 24h | Same — never delegated |
| Client communication (go-live, status, escalation) | Sebastian Puig | 24h | Esme can hold the line / acknowledge, not commit |
| Build PRs (feature, non-Protected-Path) | Sebastian Puig | 24h | Esme co-Approver for IFleet-evolution PRs |
| Spec PR (PR #1 per client) | Sebastian + Esme (both) | 48h | Both required — no proxy |

### Driver (exactly one, on our side)

- **Name:** Sebastian Puig
- **Owns:** scheduling, escalation, getting decisions made on time, ensuring the right people are consulted
- **Contact:** Discord (`@monstersebas1`) + Mac local
- **Backup driver:** Esme (when Sebastian explicitly hands off)

### Contributors (named, optional per decision)

| Name | Role | What they're consulted on |
|---|---|---|
| Esme | Builder / Tool researcher | IFleet evolution, tool selection, n8n workflows, plugin audit |
| (Voice-AI interviewer) | Discovery agent | Decisions about interview methodology, prompt design |
| (Claude orchestrator) | This Claude session | Architecture suggestions, plan synthesis, spec drafting |
| (Reviewer subagent) | Code review | Code quality, security review pre-merge |

### Informed (distribution)

| Channel / person | Receives | Frequency |
|---|---|---|
| Discord #ifleet (channel 1504120127791042631) | All Factory operational briefs | As they happen |
| Discord 8am digest (M-018, planned) | Overnight summary | Daily |
| Per-client repo PR review queue | All client-build PRs | Real-time |

### Out of scope for this stakeholder list

- **The client** is NOT a stakeholder of The Factory itself. The client is a stakeholder of their own SaaS project (separate STAKEHOLDERS.md per client repo).
- **End-users of client SaaS** are NOT stakeholders of The Factory. Same boundary.

---

## Decision log

| Date | Decision | Approver | Driver | Reference |
|---|---|---|---|---|
| 2026-05-12 | Create IFleet repo public (over $4/mo Team upgrade for private) | Sebastian | Sebastian | `[[ifleet-repo]]` memory |
| 2026-05-12 | Phase B model: sonnet default, opus on complexity:high | Sebastian | Sebastian | PR #41 in IFleet repo |
| 2026-05-13 | SearXNG MCP on Mac (self-hosted) | Sebastian | Sebastian | `[[searxng-mcp]]` memory |
| 2026-05-14 | Remove Damian from team / memory | Sebastian | Sebastian | This conversation, memory cleaned |
| 2026-05-14 | Plain-English recap mandatory after every technical explanation | Sebastian | Sebastian | `memory/feedback_plain_english_recap.md` + global CLAUDE.md |
| 2026-05-14 | Continuous-execution default (don't idle between tasks) | Sebastian | Sebastian | Global CLAUDE.md, section "Continuous execution" |
| 2026-05-15 | Build full pipeline before client #1 (no manual shortcuts) | Sebastian | Sebastian | ADR-0004 |
| 2026-05-15 | Four-repo structure (Factory + IFleet + voice-discovery + spec-template + per-client) | Sebastian | Sebastian | ADR-0002 |
| 2026-05-15 | AGENTS.md primary, CLAUDE.md as @AGENTS.md bridge | Sebastian | Sebastian | ADR-0001 |
| 2026-05-15 | Issues remain unit of work for IFleet (4 upstream sources) | Sebastian | Sebastian | ADR-0003, ADR-0010 |
| 2026-05-15 | Retell AI as voice default (pending Esme R1 confirmation) | Sebastian | Sebastian | ADR-0006 |
| 2026-05-15 | Mermaid + C4 L1-L2 only for diagrams | Sebastian | Sebastian | ADR-0007 |
| 2026-05-15 | MADR format for ADRs, append-only via filename | Sebastian | Sebastian | ADR-0008 |
| 2026-05-15 | Four-tier data classification + NIST CIA mapping | Sebastian | Sebastian | ADR-0009 |
| 2026-05-15 | Tier 3 self-heal deferred until 2 weeks production data | Sebastian | Sebastian | ADR-0005 |
| 2026-05-15 | Esme as builder, not just researcher (6 of 8 IFleet gaps) | Sebastian | Sebastian | This conversation, ROADMAP ownership column |
| 2026-05-15 | Markdown-only spec files (no `.yaml` siblings until parsing fails) | Sebastian | Sebastian | ADR-0001 reasoning, "correct not clever" |
| 2026-05-15 | Empty files use `status: skeleton` instead of fabricated content | Sebastian | Sebastian | "correct not clever" framing |

---

## Escalation paths

### If Sebastian is unreachable > 48 hours
1. Esme can acknowledge inbound briefs in Discord (act as comms holding pattern).
2. Esme proceeds with non-Protected-Path build work using last-known decisions.
3. Anything hitting a No-Go in `NON_GOALS.md` is auto-rejected without Sebastian's input needed.
4. Anything in the Protected Paths list waits — never proxy this.

### If a decision is needed urgently and Sebastian is unreachable
1. Default to the **most conservative interpretation** of existing ADRs.
2. Log the temporary decision as a new ADR with `status: proposed`.
3. Sebastian reviews on return; either accepts (flip to `accepted`) or supersedes.

### If a worker is stuck
1. Open a GitHub issue labeled `clarification-needed`.
2. State: what was tried, what's ambiguous, proposed conservative default.
3. Tag `@monstersebas1` only after 1 hour of being stuck (avoid notification spam).
4. If autonomous (ralph/autopilot), pick conservative default + log in `memory/handoff.md` + proceed.

---

**Last updated:** 2026-05-26
**Last verified:** 2026-05-26 — nightly audit (read-only review; no content changes)

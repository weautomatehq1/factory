---
adr: "0006"
title: "Retell AI as voice platform default (pending Esme R1 confirmation)"
status: accepted
date: 2026-05-15
deciders: [sebas]
consulted: [esme, voice-research]
informed: [voice-discovery]
tags: [voice, integration]
supersedes: null
---

# ADR-0006: Retell AI as voice platform default

## Context and Problem Statement

The Factory's voice interviewer needs telephony + STT + TTS + interrupt handling (barge-in) + tool-calling during the call. Several platforms exist: Retell AI, Vapi.ai, ElevenLabs Conversational AI, OpenAI Realtime API, Cartesia. Which do we commit to for the initial build?

## Decision Drivers

- Must handle 30–60 min long calls without state loss
- Must support tool-calling mid-call (4 tools: `mark_covered`, `flag_ambiguity`, `escalate_to_human`, `end_interview`)
- Must handle interrupts cleanly (barge-in)
- Must integrate with n8n (we have an OAuth-handling automation layer)
- Latency target: <800ms round-trip
- Cost-aware but not cost-driven
- An existing OMC skill `voice-ai-agent` already documents Retell + n8n + Supabase as a pattern
- Esme R1 research (per SPRINT-2026-05-A) will validate or recommend pivot

## Considered Options

1. **Retell AI** — purpose-built for AI voice agents; documented OMC pattern; n8n integration available
2. **Vapi.ai** — similar capabilities, marginally different pricing
3. **ElevenLabs Conversational AI** — strong voice quality, newer agent runtime
4. **OpenAI Realtime API** — direct WebRTC, requires us to handle telephony separately (Twilio)
5. **Cartesia** — fast TTS, lighter on agent orchestration

## Decision Outcome

Chosen: **Retell AI (default, pending Esme R1 confirmation in SPRINT-2026-05-A).**

If Esme's R1 research reveals a materially better option, this ADR is superseded by a follow-up ADR. The pick is *locked for the sprint* (per SPRINT.md § No-gos item 5: "NOT migrating from Retell if R1 research suggests Vapi is marginally cheaper. Switching mid-sprint = sprint death.").

<!-- Erratum 2026-05-27: The original text read "NON_GOALS.md NG-something equivalent" — a placeholder that was never resolved. The actual sprint-scoped constraint lives in SPRINT.md § No-gos #5. The nearest permanent analogue in NON_GOALS.md is NG-6 (model selection locked mid-sprint) whose "sprint death" rationale applies equally here. -->

### Why not the others?
- **Vapi.ai:** comparable; Retell is documented in our OMC skill so onboarding is faster. Marginal cost difference doesn't justify the documentation rebuild.
- **ElevenLabs Conversational AI:** voice quality is excellent but tool-calling + interrupt handling are less mature than Retell as of 2026-05.
- **OpenAI Realtime API:** requires us to handle telephony (Twilio) ourselves. Adds an integration we don't need.
- **Cartesia:** primarily a TTS provider; agent orchestration would be ours to build. Adds complexity.

## Consequences

- **Good:**
  - Fastest path to operational voice interviewer (skill exists, documentation available)
  - n8n integration is a known pattern
  - Locked for the sprint = no decision churn during build
- **Bad:**
  - Vendor lock-in to Retell. Mitigation: voice-discovery's prompt + tools are platform-agnostic; only the webhook adapter is Retell-specific. Swap effort: ~1 day.
  - First integration always slower than expected (RISKS R-005)
- **Neutral:**
  - Per-minute cost depends on usage; reviewed quarterly

## Confirmation

- Verify: M-005 succeeds within appetite (1 day)
- Verify: test call reaches `end_interview()` with all 4 tools callable
- Verify: Esme R1 research either confirms Retell or recommends a switch with quantified delta
- If R1 recommends switch: open new ADR superseding this one, scope = post-sprint replacement

## Related

- Related: ROADMAP [M-005] [M-006]
- Related skill: `~/.claude/skills/voice-ai-agent`
- Related: INTEGRATIONS.md "Retell AI" entry
- Related risks: R-004 (Retell tool-calling state loss), R-005 (webhook auth)

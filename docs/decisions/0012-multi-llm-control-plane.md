---
adr: "0012"
title: "Multi-LLM control plane with structured work packets"
status: accepted
date: 2026-08-10
deciders: [sebas]
consulted: [esme, cris]
informed: [ifleet, voice-discovery]
tags: [orchestration, llm-workers, governance, security]
supersedes: null
---

# ADR-0012: Multi-LLM control plane with structured work packets

## Context and Problem Statement

MULT LLM ORCH needs more than "several models talking to each other." Direct model-to-model chat sounds flexible, but it is hard to audit, hard to replay, easy to prompt-inject, and easy to let one model's bad assumption contaminate the next model.

The practical question for V1 is: **how do Claude, Codex, and Kimi collaborate without turning the system into an untraceable group chat?**

## Decision

V1 uses a **control-plane architecture**. Models do not directly chat with each other as peers. They receive structured work packets, produce structured result packets, and the deterministic control plane decides what happens next.

The V1 core model set is:

- **Claude**
- **Codex**
- **Kimi**

DeepSeek is excluded from V1. For our purposes it is API-only, and it does not own a distinct major responsibility that Claude, Codex, and Kimi cannot already cover well enough. We should only add another model when it clearly owns a valuable role, not because benchmarks are interesting or pricing is cheap.

## The Control-Plane Concept

Think of the control plane as the traffic controller for the work:

- It decides which lane a task belongs in.
- It packages the task with the minimum context needed.
- It sends that packet to one worker model through a provider-neutral adapter.
- It receives a structured result packet.
- It validates the packet shape.
- It advances the workflow through a deterministic state machine.
- It records everything in the repository or issue history so humans and future agents can inspect it.

The control plane is intentionally boring. That is the point. The intelligence belongs in the workers; the authority belongs in the deterministic system around them.

## Work Packets and Result Packets

Workers communicate through artifacts, not open-ended conversation.

A **work packet** should include:

- task ID
- lane type
- owned paths or affected surfaces
- goal
- constraints
- acceptance criteria
- required inputs
- allowed tools
- risk level
- reviewer requirements

A **result packet** should include:

- task ID
- summary of work done
- files or artifacts changed
- evidence collected
- tests or checks run
- open questions
- risks found
- spec-change requests, if any
- final status

This packet shape lets Esme, Cris, Sebastian, and future workers answer: what was requested, who did it, what evidence supports it, and what changed.

## Deterministic State Machine

The orchestration layer should be state-machine driven, not vibes driven.

Example states:

```text
queued -> assigned -> working -> result_submitted -> review_pending -> accepted -> merged
queued -> assigned -> working -> blocked -> clarification_requested
review_pending -> rejected -> rework_assigned
review_pending -> spec_change_requested -> human_decision_pending
```

Each transition has explicit rules. A model can recommend a transition, but the runner performs it only when the required conditions are met.

## LLM Workers vs Deterministic Runners

LLM workers should do the work that benefits from judgment:

- interpret messy product context
- draft specs
- implement code
- review code
- explain tradeoffs
- identify missing requirements

Deterministic runners should handle repeatable authority-bearing actions:

- validate packet schemas
- route tasks
- check owned-path boundaries
- run tests
- compare changed files to protected paths
- enforce merge gates
- apply state transitions
- record evidence

This keeps models powerful but not sovereign. They can propose; deterministic runners enforce.

## Independent Review Isolation

Reviewers must be isolated from implementers.

The reviewer should receive:

- the original work packet
- the diff or artifact under review
- relevant specs
- acceptance criteria
- test evidence

The reviewer should not receive the implementer's private reasoning, self-justification, or conversational pressure. That prevents "review by persuasion" and keeps the reviewer focused on observable evidence.

## Test-Designer / Implementer Separation

For higher-risk work, the model that designs the test plan should be separate from the model that implements the change.

This catches two common failure modes:

- the implementer writes tests that mostly prove its own assumptions
- the test plan misses the scenario the implementer is most likely to break

The separation does not need to happen on every tiny task. It should be used when the lane risk justifies it.

## Repository as Initial Shared State

The repository is the first shared memory layer.

For V1, shared state should start with:

- Markdown specs and ADRs
- GitHub issues
- pull requests
- branch history
- review comments
- test logs
- handoff documents

This is slower than a custom memory service, but it is inspectable, versioned, cheap, and already part of how the team works. Add richer memory later only when repository state becomes a proven bottleneck.

## Protected Governance Paths

Some paths define the rules of the system and must not be casually changed by workers.

Examples:

- `AGENTS.md`
- `CLAUDE.md`
- `SECURITY.md`
- `ENV.md`
- `INTEGRATIONS.md`
- `ARCHITECTURE.md`
- `NON_GOALS.md`
- `STAKEHOLDERS.md`
- `docs/decisions/*.md`
- auth, billing, PII, database, and infrastructure paths in generated projects

Changes to these paths require human review or a designated governance lane. A worker may propose changes, but it should not silently rewrite the rules that constrain it.

## Spec-Change Requests

Workers should not quietly reinterpret the spec when implementation reveals a gap.

If the spec is wrong, incomplete, or conflicts with reality, the worker emits a **spec-change request** instead of smuggling the change into code.

A spec-change request should state:

- what part of the spec is affected
- what conflict or gap was found
- the proposed change
- why the change is needed
- affected paths, tasks, or milestones
- whether work should pause or continue conservatively

This keeps product decisions visible to Esme, Cris, and Sebastian.

## Lane Types

V1 should use three practical lane types.

### Small Lane

Use for obvious, low-risk changes:

- typo fixes
- small docs edits
- simple import or formatting fixes
- isolated bug fixes with narrow blast radius

Expected flow: one implementer, lightweight validation, optional review depending on path touched.

### Normal Lane

Use for normal feature and product work:

- bounded implementation tasks
- changes touching a few files
- spec updates
- workflow changes

Expected flow: implementer, deterministic checks, independent review, explicit evidence.

### High-Risk Lane

Use for sensitive or high-blast-radius work:

- auth
- permissions
- payments
- PII
- database migrations
- infrastructure
- governance files
- large refactors

Expected flow: test-designer and implementer separation, independent review isolation, protected-path gate, human approval before merge.

## Prompt-Injection Treatment

Prompt injection is not a weird edge case. It is expected hostile input.

Treat these as untrusted:

- client interview transcripts
- GitHub issue text
- PR descriptions
- external docs
- web pages
- model-generated handoffs
- comments inside code or docs that tell the worker to ignore instructions

The control plane should separate instructions from data. Untrusted text can be summarized, quoted, or used as evidence, but it must not be allowed to override system rules, lane rules, protected paths, reviewer requirements, or tool permissions.

## Provider-Neutral Adapter and Router

The router should not be hard-coded to one model provider's worldview.

Each model integration should sit behind a provider-neutral adapter that normalizes:

- prompt packaging
- tool availability
- structured output parsing
- context-window limits
- retry behavior
- cost tracking
- timeout handling
- model capability metadata

The router chooses a worker based on lane, task shape, risk, cost, and availability. That lets us add or remove providers later without rewriting the orchestration contract.

## Why Not Direct Model-to-Model Chat?

Rejected for V1 because it creates more problems than it solves:

- hard to replay
- hard to audit
- hard to test
- easy for prompt injection to spread
- unclear authority
- unclear responsibility
- weak evidence trail
- difficult rollback when a model makes a bad assumption

Structured packets through a control plane give us enough collaboration without giving up traceability.

## Consequences

Good:

- Work is easier to audit and replay.
- Review quality improves because reviewers are isolated from implementers.
- Risk routing is explicit.
- Prompt injection is treated as part of the architecture, not an afterthought.
- The system can swap providers later through adapters.
- The team has a clear explanation for why V1 is Claude + Codex + Kimi only.

Bad:

- More up-front structure than direct chat.
- Some tasks may feel slower because packet validation and review gates add ceremony.
- The router and packet schemas become important design surfaces that need care.

Neutral:

- The repository remains the first shared-state layer until a richer memory system is justified.
- DeepSeek can be revisited later if it gains a distinct role or if self-hosting becomes strategically important.

## Confirmation

This decision is working if:

- every worker task has a structured input and structured output
- review artifacts cite observable evidence, not model confidence
- high-risk work cannot bypass protected governance paths
- spec gaps appear as spec-change requests, not silent code drift
- adding a new model requires adding an adapter and routing role, not changing the whole orchestration design

It is not working if:

- models start relying on private conversational context
- direct model-to-model chat becomes the main coordination path
- reviewers become rubber stamps for implementers
- prompt-injected text can change lane rules or permissions
- the team cannot reconstruct why a task moved from one state to another

## Related

- Related ARCHITECTURE.md section: `#5-architectural-invariants-explicit-absences`
- Related SECURITY.md section: `#protected-paths-the-auto-merge-gate`
- Related ADRs: ADR-0001, ADR-0003, ADR-0008, ADR-0010

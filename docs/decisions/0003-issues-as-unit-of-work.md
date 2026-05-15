---
adr: "0003"
title: "GitHub Issues remain the unit of work for IFleet"
status: accepted
date: 2026-05-15
deciders: [sebas]
consulted: [esme, claude-orchestrator]
informed: [ifleet]
tags: [ifleet, orchestration]
supersedes: null
---

# ADR-0003: GitHub Issues remain the unit of work for IFleet

## Context and Problem Statement

IFleet's current trigger model: a human files a GitHub Issue → SprintManager spawns a worker → worker picks the issue → ships a PR. With the addition of the Decomposer (M-002) and Sentry/PostHog auto-issue bridges, should we keep "Issues are the atomic unit of work" as the contract, or move to a richer task model (custom queue, spec-section-as-task, etc.)?

## Decision Drivers

- Workers need atomic claim (no two workers grabbing the same task)
- Branch protection at GitHub requires PR-to-issue linkage (we already use this)
- Humans want to see what's queued without spinning up a custom dashboard
- Humans want to edit/close a task mid-sprint (intervene easily)
- Self-healing needs to enqueue work from Sentry/PostHog
- Multiple upstream sources should not require multiple queue implementations

## Considered Options

1. **Direct spec execution** — workers read `ROADMAP.md` / `SPRINT.md` directly, claim sections via a custom queue
2. **Issues as queue** — every upstream source creates a GitHub Issue; workers grab next unblocked
3. **Hybrid** — Issues for human-visible work, custom queue for self-heal events

## Decision Outcome

Chosen: **Option 2 — Issues as queue, period.**

Every source of work creates a GitHub Issue:
- Spec PR merges → Decomposer (M-002) creates N issues
- Sentry error / PostHog anomaly → Sentry-n8n bridge creates 1 issue
- Failed deploy → Deploy worker (M-012) creates 1 issue
- Cron failure → monitoring hook creates 1 issue
- Humans → file directly

Workers pick the next unblocked issue (DAG-aware per M-003 dependency-aware queue).

### Why not the others?
- **Option 1 (direct spec execution):** building our own queue means rebuilding atomic claim, permissions, observability, and intervention UX. GitHub Issues already gives us all four. Path of "build it ourselves" is documented anti-pattern.
- **Option 3 (hybrid):** two queues means two integration patterns to maintain. Anytime we add a new upstream source, we'd have to decide which queue it talks to. Premature optimization for a problem we don't have.

## Consequences

- **Good:**
  - One universal interface for IFleet (Issue created → worker reacts)
  - Free observability via GitHub Issues UI — Sebastian and Esme see the queue
  - Easy human intervention (close issue, edit description, add label)
  - Branch protection + assignee mechanics already designed for this
  - Sentry → n8n → GitHub Issue bridge already exists (per `[[project-status-20260513]]`)
- **Bad:**
  - We're coupled to GitHub. If GitHub Issues changes or rate-limits us materially, we're affected. Mitigation: rate-limit awareness in IFleet workers (in RISKS R-006).
  - Issues created by automation (Decomposer, Sentry bridge) can be voluminous. Mitigation: label `auto-generated`, dependency-aware queue prevents bulk pickup.
- **Neutral:**
  - We do not get fancy queue features (priority math, weighted fair-share) for free. Acceptable for current scale (1–3 clients).

## Confirmation

- Verify: Decomposer-created issues are picked up by workers (M-002 acceptance criteria)
- Verify: Sentry → GitHub auto-issue bridge produces an issue within 60s of a Sentry event (after M-017 ships)
- Verify: human can close an auto-issue and worker doesn't re-create it within the same session
- Periodic: are we hitting GitHub API rate limits? (RISKS R-006 monitors)

## Related

- Related ARCHITECTURE.md section: `#5-architectural-invariants` (item 3)
- Related: ADR-0010 (the four upstream sources), ROADMAP [M-002] [M-003]

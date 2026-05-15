---
adr: "0002"
title: "Four-repo structure: Factory + IFleet + voice-discovery + spec-template + per-client"
status: accepted
date: 2026-05-15
deciders: [sebas]
consulted: [esme, claude-orchestrator]
informed: [ifleet, voice-discovery, spec-template]
tags: [architecture, repos]
supersedes: null
---

# ADR-0002: Four-repo structure for The Factory

## Context and Problem Statement

The Factory is a system that builds SaaS products. The system itself comprises multiple distinct concerns (interview, spec, build, deploy, monitor). How do we organize the source code: monorepo? polyrepo? what's the boundary?

## Decision Drivers

- IFleet already exists as a standalone repo (`github.com/weautomatehq1/IFleet`)
- Voice-discovery is a separate concern with a different deploy model (n8n + Retell, not Vercel)
- Per-client repos must be isolatable — one client's repo cannot import another's
- spec-template must be a GitHub template (a single repo type that can be "Used as template")
- IFleet workers must be able to commit to client repos without modifying their own repo from a client task
- Branch protection rules apply per-repo (we want IFleet's branch protection separate from client repos')
- Cost: 4–5 repos at GitHub Free is free (with branch protection on public repos only — see [[ifleet-repo]])

## Considered Options

1. **Monorepo** — single repo containing `factory/`, `ifleet/`, `voice-discovery/`, `templates/spec/`, and per-client folders
2. **Two-repo split** — `weautomatehq1/factory` (everything Factory-side) + per-client repos
3. **Four-repo structure** — separate repos for Factory + IFleet + voice-discovery + spec-template; per-client repos created from spec-template

## Decision Outcome

Chosen: **Option 3 — four-repo structure.**

The boundaries:
- `weautomatehq1/factory` (this repo) — master coordination, 17 spec files describing The Factory itself, ADRs
- `weautomatehq1/IFleet` — autonomous worker fleet (exists; Phase A green 2026-05-13)
- `weautomatehq1/voice-discovery` — voice interviewer, n8n workflows, synthesizer (M-005)
- `weautomatehq1/spec-template` — GitHub template repo (M-001)
- `weautomatehq1/<client-name>` — one per client, spawned from spec-template

### Why not the others?
- **Monorepo:** branch protection in GitHub Free applies per-repo, not per-folder. We'd lose granular protection. Also: IFleet workers checking out the whole monorepo to fix a client bug would be unnecessarily heavy.
- **Two-repo:** collapsing voice-discovery into factory mixes a markdown-only spec repo with a TypeScript+n8n deploy target. Cognitively muddier. Voice-discovery has its own deploy lifecycle (Retell config changes, n8n workflow versions) that shouldn't bump the spec repo's CHANGELOG.

## Consequences

- **Good:**
  - Clean separation of concerns; each repo has one job
  - Per-repo branch protection works
  - IFleet workers can clone JUST what they need
  - Per-client repos are fully isolated (no path-based access to other clients' code)
  - Failure in one repo (e.g., voice-discovery broken) doesn't cascade to others
- **Bad:**
  - 4 base repos to maintain (plus N per-client). Some duplicated config (eslint, tsconfig) — acceptable.
  - Cross-repo refactors require synchronized PRs in multiple repos. Mitigated by Decomposer (M-002) emitting per-repo issues with shared `epic:` label.
  - More GitHub orgs cluttering Sebastian's view. Acceptable.
- **Neutral:**
  - Esme and Sebastian both need clone-access to all repos. Already the case via weautomatehq1 org membership.

## Confirmation

- Verify: a worker fixing an IFleet bug doesn't accidentally touch voice-discovery files (proven by separate repo cloning)
- Verify: a security incident in one client repo doesn't expose other clients' code (proven by repo-level RLS at GitHub)
- Quarterly check: is the four-repo overhead causing PR-coordination pain? If yes, revisit (likely supersede with a sub-tree merge plan).

## Related

- Related ARCHITECTURE.md section: `#3-container-view-c4-level-2`
- Related: `[[ifleet-repo]]` memory; ROADMAP [M-001], [M-005]

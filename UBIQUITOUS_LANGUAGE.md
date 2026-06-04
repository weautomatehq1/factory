---
file: UBIQUITOUS_LANGUAGE.md
status: accepted
last_updated: 2026-05-26
owner: sebas
---

# Ubiquitous Language

> Domain vocabulary. The term in the **Canonical** column is the only one allowed in code, UI, comments, commits, and conversation. If a synonym appears in the wild, replace it. Client's word wins in client repos; The Factory's terms below apply to this repo and the system itself.

## Discipline rules

1. The canonical term is the **only** allowed name. Aliases listed are banned, not just discouraged.
2. Add new terms only when the client (or, for The Factory's own terms, Sebastian) uses one. **Do not invent terms.**
3. Past-tense events (per Event Storming) — record them in past tense: `SpecPRMerged`, `WorkerCompleted`, `InterviewEnded`. These become event names in code.
4. Voice interviewer rule: echo a term back to confirm understanding before continuing. "When you say *matter*, do you mean X or Y?"
5. Cite source for every term: `INTAKE YYYY-MM-DD [HH:MM]` or `ADR-NNNN` or `Sebastian 2026-MM-DD conversation`.

## Bounded contexts

The Factory operates across four bounded contexts. The same word can mean different things in different contexts; document the boundary.

- **Discovery context** — the voice interview phase. Terms: Interview, Client, Coverage, Transcript, Ambiguity.
- **Spec context** — the 17-file phase. Terms: Spec, Milestone, Sprint, ADR, Invariant.
- **Build context** — the IFleet execution phase. Terms: Worker, Issue, PR, Gap, Decomposer, Synthesizer.
- **Operate context** — post-launch phase. Terms: KPI, Tier, Killswitch, Protected Path, Self-Heal, Postmortem.

---

## Glossary

### Factory
- **Canonical:** The Factory (capitalized, with "The")
- **Context:** all
- **Definition:** WeAutomateHQ's autonomous client-SaaS factory. Voice interview → spec → autonomous build → self-heal. The system as a whole.
- **NOT:** "the system," "the platform," "the pipeline," "the factory" (lowercase), "OMF," "WAHQ-Factory"
- **Source:** Sebastian 2026-05-12 conversation

### IFleet
- **Canonical:** IFleet (one word, capital I, capital F)
- **Context:** Build
- **Definition:** The autonomous fleet of Claude Code Pro Max + Codex Pro workers that picks up GitHub Issues, runs in isolated git worktrees, and ships PRs. Lives at `~/dev/ai-products/IFleet`.
- **NOT:** "the fleet" (lowercase, alone), "iFleet," "I-Fleet," "Claude fleet," "worker pool" (worker pool is a component of IFleet, not IFleet itself)
- **Source:** Sebastian 2026-05-12 conversation, [[ifleet-repo]] memory

### Worker
- **Canonical:** Worker
- **Context:** Build
- **Definition:** A single Claude or Codex process spawned by SprintManager that picks one GitHub Issue, runs the architect → editor → reviewer pipeline, and opens a PR. Lives in an isolated git worktree.
- **NOT:** "agent" (agent is broader — see below), "bot," "process," "job"
- **Related:** Architect (planner role), Editor (code-writing role), Reviewer (review role) — these are *roles within a worker*, not separate workers
- **Source:** [[ifleet-architecture]] memory

### Agent
- **Canonical:** Agent
- **Context:** all
- **Definition:** Any AI system reading `AGENTS.md` and acting in a repo. Includes Claude Code, Codex, Cursor, Aider, Amp, Jules. Broader than "Worker" — workers are agents, but so is Sebastian's interactive Claude session.
- **NOT:** "AI," "bot," "LLM" (these are model-tier terms; agent is system-tier)
- **Source:** agents.md spec convention

### Decomposer
- **Canonical:** Decomposer (capital D)
- **Context:** Build
- **Definition:** The IFleet component (Gap 1, M-002) that reads merged `ROADMAP.md` + `ARCHITECTURE.md` and emits dependency-ordered GitHub Issues. Each issue cites its source spec section via `spec: <file>#<anchor>`.
- **NOT:** "issue generator," "decomposer agent" (it's a component, not a standalone agent), "spec parser" (parsing is a sub-step, not the whole responsibility)
- **Source:** ADR-0003, ROADMAP [M-002]

### Synthesizer
- **Canonical:** Synthesizer (capital S)
- **Context:** Discovery → Spec transition
- **Definition:** The Claude Opus pipeline (M-010) that reads a Supabase-stored transcript + coverage map and emits the 17 spec files into a new GitHub repo created from spec-template.
- **NOT:** "spec generator" (synthesizer is post-interview only), "compiler"
- **Source:** ROADMAP [M-010]

### Spec
- **Canonical:** Spec
- **Context:** all
- **Definition:** The set of 17 markdown files at a project root that define what to build. Lives in The Factory repo (this) for The Factory itself, in spec-template for the template, and in `weautomatehq1/<client-name>` for a client project.
- **NOT:** "the docs" (docs is broader), "requirements," "PRD" (PRDs are a single doc; spec is plural+structured)
- **Related:** Spec PR (the first PR in a client repo that lands the synthesized spec)
- **Source:** Sebastian 2026-05-14 conversation

### Interview
- **Canonical:** Interview
- **Context:** Discovery
- **Definition:** A single phone-based discovery session between the voice agent and the client. Duration target 30–60 min. Ends when coverage ≥ 80% on required spec sections AND the client has been asked "what haven't we talked about?"
- **NOT:** "discovery call" (call is the medium, not the activity), "session" (too vague)
- **Lifecycle:** scheduled → in-progress → ended → transcribed → synthesized
- **Source:** Sebastian 2026-05-14 conversation

### Coverage
- **Canonical:** Coverage
- **Context:** Discovery
- **Definition:** The weighted average of section-fill confidence across required + optional spec sections during an interview. Layer 2 of the voice agent tracks coverage silently. End-of-interview gate: coverage ≥ 0.8 on required, ≥ 0.6 on optional.
- **NOT:** "completeness," "progress"
- **Source:** ROADMAP [M-006]

### Layer 1 / Layer 2
- **Canonical:** Layer 1 (therapy mode), Layer 2 (coverage controller)
- **Context:** Discovery
- **Definition:** The two simultaneous controllers running inside the voice interviewer. Layer 1 is the visible therapy-mode conversation (empathetic, open-ended, reflective). Layer 2 is the silent coverage tracker that scores section-fill and redirects when needed.
- **NOT:** "outer layer / inner layer" (ambiguous), "modes" (they're not modes, they run together)
- **Source:** Sebastian 2026-05-14 conversation

### Gap
- **Canonical:** Gap (capital G when ID'd, e.g., "Gap 1")
- **Context:** Build (IFleet evolution)
- **Definition:** One of the 8 missing IFleet components needed for spec → deployed: Decomposer (1), DAG Queue (2), Bootstrapper (3), Schema Worker (4), Policy Hook (5), Design Bootstrap (6), Deploy Worker (7), Staging URL (8). Gap numbers are logical IDs, not build-sequence order — see ROADMAP.md for execution order (M-002=Gap1, M-003=Gap2, M-004=Gap5, M-008=Gap4, M-009=Gap6, M-014=Gap3, M-012=Gap7, M-013=Gap8).
- **NOT:** "feature" (gap is specifically a missing piece, not a new feature)
- **Source:** Sebastian 2026-05-14 conversation, ROADMAP [M-002 to M-014]

### Tier
- **Canonical:** Tier (capital T when ID'd, e.g., "Tier 1")
- **Context:** Operate (self-healing)
- **Definition:** One of the three self-healing layers. Tier 1 = build-time (CI failure → intervention). Tier 2 = deploy-time (smoke fail → rollback). Tier 3 = runtime (Sentry/PostHog → draft PR with proposed fix).
- **NOT:** "level," "stage"
- **Source:** ADR-0005, ROADMAP [M-011, M-017]

### Killswitch
- **Canonical:** Killswitch (one word)
- **Context:** Operate
- **Definition:** An environment variable that instantly disables a self-healing tier or other autonomous behavior. Examples: `SELF_HEAL_BUILD=off`, `SELF_HEAL_DEPLOY=off`, `SELF_HEAL_RUNTIME=off`. Honored by all workers; flipping = effective within the next worker tick.
- **NOT:** "kill switch" (two words), "off-switch," "emergency stop"
- **Source:** ADR-0005

### Protected Path
- **Canonical:** Protected Path (capital P, capital P)
- **Context:** Operate
- **Definition:** A file path glob listed in `SECURITY.md#protected-paths` that auto-merge MUST NEVER touch. Includes auth, billing, PII, infrastructure code. Enforced by the auto-merge gate; bypassable only via human merge.
- **NOT:** "sensitive path," "guarded path"
- **Source:** ARCHITECTURE.md §5 (invariant 5), SECURITY.md §Protected-Paths

### Operator
- **Canonical:** Operator (capital O)
- **Context:** all
- **Definition:** The human(s) running The Factory: Sebastian (Driver + Approver) and Esme (Builder + Contributor). NOT the client.
- **NOT:** "user," "admin," "developer," "engineer"
- **Source:** STAKEHOLDERS.md

### Client
- **Canonical:** Client (capital C)
- **Context:** Discovery
- **Definition:** The person buying a SaaS from WeAutomateHQ. Goes through one voice interview. Reviews staging URLs. Does not write code or merge PRs.
- **NOT:** "user" (user is the eventual end-user of the client's SaaS, not the client themselves), "customer," "stakeholder"
- **Lifecycle:** lead → interviewed → spec-approved → onboarded → active → churned/expanded
- **Source:** Sebastian 2026-05-14 conversation

### Issue
- **Canonical:** Issue (capital I when referring to GitHub Issue specifically)
- **Context:** Build
- **Definition:** A GitHub Issue. The atomic unit of work for IFleet workers. Per ADR-0003, issues are the universal interface — created by 4 upstream sources (decomposer, Sentry bridge, deploy worker, cron monitor) plus humans.
- **NOT:** "ticket," "task," "story" (these are agile-tool terms; we're GitHub-native)
- **Source:** ADR-0003

### Spec PR
- **Canonical:** Spec PR (or "PR #1" when there's no ambiguity)
- **Context:** Discovery → Build transition
- **Definition:** The first pull request opened in a client repo, containing all 17 synthesized spec files. Approval of this PR by both operators is the gate that triggers the decomposer.
- **NOT:** "initial PR," "setup PR"
- **Source:** ROADMAP [M-010]

### Sprint
- **Canonical:** Sprint (capital S; reference by ID, e.g., "SPRINT-2026-05-A")
- **Context:** Spec, Build
- **Definition:** A 1–2 week cycle of work, defined in `SPRINT.md`. Adapted Shape Up: Problem, Appetite (hard time-cap), Solution sketch, Rabbit holes, No-gos, Tasks, Done when. One sprint maps to one or more `[M-NNN]` milestones.
- **NOT:** "iteration," "cycle" (cycle is broader)
- **Source:** Sebastian 2026-05-15 conversation; Shape Up (Basecamp) adapted for 2-person team — no formal ADR written, decision considered self-evident. ADR-0008 covers ADR format, not sprint methodology.

### Milestone
- **Canonical:** Milestone (referenced by ID, `[M-NNN]`)
- **Context:** Spec
- **Definition:** A roadmap-level unit of work with success criteria, owner, appetite, and dependencies. Listed in `ROADMAP.md`. Bracketed ID format is REQUIRED so the decomposer can regex-match.
- **NOT:** "epic," "phase" (phase is broader)
- **Source:** ROADMAP.md format spec

### ADR
- **Canonical:** ADR (uppercase, referenced by `ADR-NNNN`)
- **Context:** Spec
- **Definition:** Architectural Decision Record. One per file at `docs/decisions/NNNN-kebab-title.md`. MADR format. Append-only via filename discipline.
- **NOT:** "decision doc," "RFC" (RFC is proposal-stage; ADR is decided-stage)
- **Lifecycle:** proposed → accepted | rejected | superseded by NNNN
- **Source:** ADR-0008

### Voice Discovery
- **Canonical:** voice-discovery (lowercase, hyphenated — repo name)
- **Context:** all
- **Definition:** The repo that contains the Retell config, n8n workflow, Layer 1 + Layer 2 prompts, operator dashboard, and synthesizer. Created M-005.
- **NOT:** "voice agent," "voice AI" (these describe the function, not the repo)
- **Source:** ROADMAP [M-005]

### Spec Template
- **Canonical:** spec-template (lowercase, hyphenated — repo name)
- **Context:** all
- **Definition:** The GitHub template repo that every client project starts from. Contains 17 file skeletons with frontmatter and stub content. Created M-001.
- **NOT:** "template repo" (too generic), "starter"
- **Source:** ROADMAP [M-001]

### Self-Heal
- **Canonical:** self-heal (verb) / self-healing (adj)
- **Context:** Operate
- **Definition:** The system attempting to fix a failure automatically before pinging a human. Three tiers (1=build, 2=deploy, 3=runtime). Bounded by killswitches and Protected Paths.
- **NOT:** "auto-fix" (auto-fix is mechanical; self-heal includes diagnosis + verification), "recovery"
- **Source:** ADR-0005

### Coverage Heatmap
- **Canonical:** Coverage Heatmap
- **Context:** Discovery
- **Definition:** The visual on the operator dashboard showing which spec sections are filled during a live interview, and at what confidence level. Drives the "Interrupt" decision.
- **NOT:** "progress bar," "completeness view"
- **Source:** ROADMAP [M-007]

### PITCH
- **Canonical:** PITCH (all-caps filename)
- **Context:** Spec → Build transition
- **Definition:** A document created post-interview that proposes feature ideas authored by The Factory team, not the client. Lives in the spec repo root alongside INTAKE.md. Separate from INTAKE because INTAKE is canonical evidence (client words verbatim); PITCH is synthesis + optionality authored by The Factory.
- **NOT:** "feature request," "backlog item" (those live in GitHub Issues after PITCH content is reviewed)
- **Source:** INTAKE.md §Out-of-scope inputs

### MADR
- **Canonical:** MADR (acronym, all-caps)
- **Context:** Spec, all repos
- **Definition:** Markdown Architectural Decision Record. A template and format for writing ADRs using YAML frontmatter and a structured Markdown body. Defined at adr.github.io/madr/. All Factory ADRs in `docs/decisions/*.md` use MADR format.
- **NOT:** "ADR format" (MADR is one format among many; use "MADR format" to be specific), "decision doc"
- **Source:** ADR-0008

---

## Events (past-tense, for Event Storming → code)

When the system needs to emit/observe these events, use these exact names:

- `InterviewScheduled`
- `InterviewStarted`
- `InterviewEnded`
- `TranscriptPersisted`
- `CoverageReached` (paired with section name)
- `AmbiguityFlagged` (paired with topic)
- `SpecSynthesized`
- `SpecRepoCreated`
- `SpecPROpened`
- `SpecPRMerged`
- `MilestoneDecomposed`
- `IssueOpened`
- `WorkerClaimed` (paired with worker ID + issue ID)
- `WorkerCompleted`
- `PROpened`
- `PRMerged`
- `DeploySucceeded`
- `DeployFailed`
- `SmokeTestFailed`
- `RollbackFired`
- `SelfHealTier1Fired`
- `SelfHealTier2Fired`
- `SelfHealTier3Fired`
- `KillswitchToggled`
- `KPIRegressionDetected`
- `PostmortemOpened`

---

**Last updated:** 2026-05-26
**Last verified:** 2026-05-26 — nightly audit (read-only review; no content changes)

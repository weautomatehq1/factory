---
file: ARCHITECTURE.md
status: accepted
last_updated: 2026-05-26
owner: sebas
related_adrs: [ADR-0001, ADR-0002, ADR-0003, ADR-0007, ADR-0010]
---

# Architecture

> Blend of matklad's pragmatic codemap + C4 Levels 1–2 (Context + Container) + arc42's invariants and cross-cutting sections. Mermaid only; no toolchain.

## 1. Problem & Goals

**Problem:** building custom SaaS for clients is slow because the operator (Sebastian) does both discovery and implementation. Discovery quality (asking the right questions, capturing real context) caps implementation quality. Manual context-capture leaks. Manual implementation is parallel-blocked by the human.

**Goal:** decouple discovery from implementation. Use AI to (a) run higher-quality discovery than the operator can do live, (b) translate discovery into a complete spec, (c) build from spec with autonomous workers, (d) self-heal during build and after launch.

**Non-Goal:** see `NON_GOALS.md`. The Factory is internal agency leverage, not a productized dev tool.

Related: see `ROADMAP.md`, `NON_GOALS.md#no-gos`.

## 2. System Context (C4 Level 1)

```mermaid
flowchart LR
    Client[Client<br/>person] -->|phone call| Retell[Retell AI<br/>voice + STT + TTS]
    Retell -->|webhook| n8n[n8n<br/>weautomatehq.cloud]
    n8n -->|persist| Supabase[(Supabase<br/>transcripts + coverage)]
    n8n -->|trigger synth| Opus[Claude Opus<br/>spec synthesizer]
    Opus -->|create repo + PR| GH[GitHub<br/>weautomatehq1 org]
    GH -->|merge spec PR| IFleet[IFleet<br/>worker pool]
    IFleet -->|build, ship, deploy| Vercel[Vercel<br/>staging + prod]
    IFleet -->|errors| Sentry[Sentry]
    IFleet -->|analytics| PostHog[PostHog]
    Sentry -->|n8n bridge| GH
    Operator[Operator<br/>Sebastian + Esme] -->|approve PRs| GH
    Operator <-->|Discord #ifleet| IFleet
```

**External systems:**
- **Retell AI** — voice + telephony (auth: API key)
- **n8n** (self-hosted at weautomatehq.cloud) — orchestration
- **Supabase** — transcripts, coverage tracking, per-client app data
- **Claude Opus** — synthesizer brain
- **GitHub** — code + spec storage + Issues queue (the unit of work — see ADR-0003)
- **Vercel** — preview deploys + production hosting
- **Sentry** + **PostHog** — error tracking + analytics (feeds Tier 3 self-healing)
- **Hostinger** — VPS for any non-Vercel surfaces + DNS API
- **Stripe** — billing for client SaaS (when applicable)
- **Documenso** (sign.weautomatehq.cloud) — e-signing for client contracts

**Personas:**
- **Client** — buys a SaaS from WeAutomateHQ. Interacts with The Factory only via the voice interview.
- **Operator** — Sebastian (Driver + Approver) and Esme (Builder + Contributor). See `STAKEHOLDERS.md`.

Related: see `INTEGRATIONS.md` for auth model per service.

## 3. Container View (C4 Level 2)

```mermaid
flowchart TB
    subgraph factory[" Factory (this repo) "]
        F1[17 spec files<br/>markdown]
        F2[docs/decisions/<br/>ADRs]
        F3[.omc/STATUS.md<br/>operational state]
    end

    subgraph voice[" voice-discovery (new repo) "]
        V1[Retell config<br/>+ system prompt]
        V2[n8n workflow:<br/>webhook → Supabase]
        V3[Operator dashboard<br/>Next.js 15]
        V4[Synthesizer<br/>Opus pipeline]
    end

    subgraph template[" spec-template (new repo) "]
        T1[17 file skeletons<br/>+ frontmatter]
        T2[GitHub template config]
    end

    subgraph ifleet[" IFleet (existing) "]
        I1[SprintManager<br/>orchestrator]
        I2[Queue bridge<br/>GitHub I/O]
        I3[Worker pool<br/>Claude + Codex]
        I4[Architect → Editor → Reviewer<br/>pipeline]
        I5[Decomposer<br/>NEW: M-002]
        I6[Schema worker<br/>NEW: M-008]
        I7[Design bootstrap<br/>NEW: M-009]
        I8[Deploy worker<br/>NEW: M-012]
        I9[Policy hook<br/>NEW: M-004]
    end

    subgraph client[" weautomatehq1/<client-name> (per-client) "]
        C1[Generated from<br/>spec-template]
        C2[Built by IFleet]
        C3[Deployed to Vercel<br/>+ Hostinger]
    end

    factory -.->|defines patterns for| template
    voice -->|generates| client
    template -->|spawned per client| client
    ifleet -->|builds| client
    factory -.->|coordinates| ifleet
```

**Containers / repos:**

| Repo | Tech | Responsibility | Talks to |
|---|---|---|---|
| `factory` (this) | Markdown only | Master spec + ADRs + status | All others (read-only) |
| `voice-discovery` | Next.js 15 + n8n + Supabase | Run interviews, synthesize spec PR | Retell, Supabase, GitHub |
| `spec-template` | Markdown only | GitHub template repo | None — copied per client |
| `IFleet` | TypeScript + tsx | Build robot army | GitHub, client repos, Sentry, n8n |
| `<client-name>` | Next.js 15 + Supabase + Stripe etc. | The actual client SaaS | Per client's INTEGRATIONS.md |

## 4. Codemap (top-level folders, this repo)

```
factory/
├── AGENTS.md                  # AI worker rulebook (primary; cross-vendor)
├── CLAUDE.md                  # Claude bridge via @AGENTS.md import
├── README.md                  # human orientation
├── ROADMAP.md                 # [M-NNN] milestones with depends-on DAG
├── ARCHITECTURE.md            # this file
├── SPRINT.md                  # current active sprint
├── FILE_STRUCTURE.md          # directory + naming conventions
├── UBIQUITOUS_LANGUAGE.md     # domain glossary
├── INTAKE.md                  # raw client interview (skeleton at template stage)
├── NON_GOALS.md               # explicit out-of-scope
├── STAKEHOLDERS.md            # DACI decision rights
├── RISKS.md                   # FMEA risk register
├── KPI.md                     # success metrics (skeleton at template stage)
├── RUNBOOK.md                 # ops playbook (skeleton at template stage)
├── ENV.md                     # env vars (names + sources, NEVER values)
├── INTEGRATIONS.md            # third-party services + auth
├── SECURITY.md                # threat model + protected paths
├── DECISIONS.md               # auto-generated ADR index
├── CHANGELOG.md               # Keep a Changelog + SemVer
├── docs/
│   └── decisions/
│       ├── _template.md       # MADR format
│       └── NNNN-*.md          # one ADR per file
└── .omc/
    ├── STATUS.md              # Done / In flight / Up next
    └── costs.json             # usage log
```

## 5. Architectural Invariants (explicit absences)

These are the rules that don't show up by reading code — they show up by violations breaking the system.

1. **The Factory repo contains no executable code.** Markdown only. Code lives in IFleet / voice-discovery / per-client repos.
2. **The 17 spec files are the contract.** If code contradicts a spec, the spec wins. Open an issue; do not silently drift.
3. **Issues are the unit of work for IFleet.** Voice interview, self-healing, monitoring, humans — all upstream sources go through "create issue" → "worker picks up." See ADR-0003.
4. **No worker self-merges.** A human or designated reviewer agent merges every PR. Even green CI does not authorize self-merge.
5. **Auth / billing / PII paths are always human-merge.** Listed in `SECURITY.md#protected-paths`. The auto-merge gate enforces this regardless of confidence.
6. **The client's word always wins in `UBIQUITOUS_LANGUAGE.md`.** If client says "matter," we use "matter" everywhere. Aliases are banned, not just discouraged.
7. **ADRs are append-only via filename discipline.** One per file at `docs/decisions/NNNN-*.md`. Supersede via new file with `supersedes:` frontmatter. Never edit accepted ones.
8. **INTAKE.md stays raw forever.** Synthesis happens elsewhere. INTAKE is the evidence bucket so future agents can re-derive synthesis when models improve.
9. **No `.yaml` siblings of spec files unless markdown parsing has demonstrably failed.** Premature sync layer creates drift. See ADR-0001 reasoning.
10. **Cross-cutting concerns (auth, error handling, logging, RLS) are AGENTS.md-enforced patterns, not freestyle.** Policy hook (M-004) catches drift before merge.

## 6. Cross-Cutting Concerns

| Concern | Where it lives | Owner |
|---|---|---|
| **Authentication** | per-client repo `lib/auth/`; protected path | `SECURITY.md` |
| **Multi-tenant isolation** | Supabase RLS policies; `lib/api-guard/` | `supabase-multitenant` skill |
| **Logging** | Sentry breadcrumbs; structured logs to stdout | `RUNBOOK.md` |
| **Error handling** | API boundary returns `{ data, error, status }`; never expose stack traces | `AGENTS.md` §6 |
| **Observability** | Sentry + PostHog + n8n execution history + Session Report plugin | `KPI.md` |
| **Cost tracking** | per-session in `.omc/costs.json`; daily aggregate in Discord digest | `M-019` |
| **Secrets management** | 1Password vault → env vars at deploy; names only in `ENV.md` | `SECURITY.md` |
| **Killswitches** | env vars `SELF_HEAL_BUILD`, `SELF_HEAL_DEPLOY`, `SELF_HEAL_RUNTIME` | per-client `RUNBOOK.md` |

## 7. Boundaries & Layers

### Facade pattern (PostHog-inspired)
Each "feature module" in a client repo exposes a single `facade.ts` file. Other features import **only** from the facade. Internal implementation is invisible to siblings.

```
features/billing/
├── api/
├── components/
├── lib/
├── schemas/
└── facade.ts        ← ONLY public export. Other features import from here.
```

### Inter-repo boundaries
| Boundary | Rule |
|---|---|
| `factory` → others | Reference only (docs cite repos by name, never code-import) |
| `voice-discovery` → `spec-template` | Reads as a remote source-of-truth; commits new repos from it |
| `spec-template` → client repos | Template-only relationship (copied at creation, then independent) |
| `IFleet` → client repos | Workers commit to client repos; never modify own repo from a client task |
| Client repo → `IFleet` | Never (no upward dependency) |

### Decomposer ↔ ROADMAP contract
The decomposer's parser is the only consumer of ROADMAP.md's `[M-NNN]` blocks. Changing the block format = breaking change. Bump format version in frontmatter if altering.

## 8. Glossary

See `UBIQUITOUS_LANGUAGE.md`. Do not duplicate definitions here.

Quick pointers:
- **Factory, IFleet, Decomposer, Synthesizer, Spec, Worker, Tier, Gate, Killswitch, Protected Path** — all defined in UBIQUITOUS.

---

**Last updated:** 2026-05-26
**Last verified:** 2026-05-26 — nightly audit (read-only review; no content changes)

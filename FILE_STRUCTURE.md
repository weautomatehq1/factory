---
file: FILE_STRUCTURE.md
status: accepted
last_updated: 2026-05-26
owner: sebas
related_adrs: [ADR-0001, ADR-0002]
---

# File Structure

> How files are organized in The Factory and in repos generated from this template. AI workers consult this file to know where new code/docs go. If you're not sure where to put a file, the answer is here.

## Convention

- **One responsibility per file.** If you can describe a file with "and," split it.
- **One exported function/component per file.** File is named after the export.
- **Feature folders are self-contained.** No cross-feature internal imports — go through the feature's `facade.ts`.
- **Files > 300 lines must be split** into a folder with named children.
- **Named exports only.** No default exports.
- **kebab-case** for folders, **PascalCase** for component files, **camelCase** for utility files.

## Tree (The Factory repo, this one)

```
factory/
├── AGENTS.md
├── CLAUDE.md
├── README.md
├── ROADMAP.md
├── ARCHITECTURE.md
├── SPRINT.md
├── FILE_STRUCTURE.md           # this file
├── UBIQUITOUS_LANGUAGE.md
├── INTAKE.md
├── NON_GOALS.md
├── STAKEHOLDERS.md
├── RISKS.md
├── KPI.md
├── RUNBOOK.md
├── ENV.md
├── INTEGRATIONS.md
├── SECURITY.md
├── DECISIONS.md                 # auto-generated index
├── CHANGELOG.md
├── docs/
│   └── decisions/
│       ├── _template.md         # MADR template
│       └── NNNN-*.md            # one ADR per file
└── .omc/
    ├── STATUS.md                # Done / In flight / Up next
    └── costs.json               # usage log
```

## Tree (template for client repos — applies to per-client SaaS, NOT this repo)

```
<client-name>/
├── [17 spec files at root]      # same as Factory, populated for client
├── docs/
│   └── decisions/
├── apps/
│   └── web/                     # Next.js 15 App Router
│       ├── app/
│       │   ├── (marketing)/     # public-facing, no auth
│       │   ├── (app)/           # authenticated
│       │   │   └── [workspace]/ # multi-tenant scope
│       │   ├── api/             # route handlers
│       │   └── layout.tsx
│       ├── components/ui/       # shadcn / shared components
│       └── lib/                 # auth, db client, logging
├── features/                    # self-contained feature modules
│   └── <feature>/
│       ├── api/                 # server actions + route.ts
│       ├── components/          # React for this feature only
│       ├── lib/                 # pure functions
│       ├── schemas/             # Zod
│       └── facade.ts            # ★ ONLY public export
├── packages/
│   ├── db/                      # Supabase migrations + types
│   │   ├── migrations/
│   │   └── types.generated.ts
│   ├── ui/                      # truly shared components
│   └── config/                  # eslint, tsconfig
├── n8n/                         # exported workflow JSON
├── memory/
│   ├── handoff.md               # cross-agent shared state
│   ├── decisions/               # ephemeral exploratory decisions
│   └── ghost-bugs.md            # unreproducible bug log
└── .github/
    ├── workflows/               # CI
    └── CODEOWNERS               # protected paths route to humans
```

## "Where do I put...?"

### In The Factory (this repo)

| If you're adding... | Put it in... | Naming |
|---|---|---|
| A new spec section | Existing spec file (don't create new top-level) | n/a |
| A new ADR | `docs/decisions/NNNN-kebab-title.md` | sequential, never reused |
| An update to a spec file's status | Frontmatter `status:` field | n/a |
| An ops note for next session | `.omc/STATUS.md` "Up next" section | n/a |
| A cost log entry | `.omc/costs.json` | append-only JSON |
| A new file template (e.g., POSTMORTEM_TEMPLATE.md) | `docs/templates/` | UPPER_SNAKE_CASE |

### In a client repo

| If you're adding... | Put it in... | Naming |
|---|---|---|
| A new page | `apps/web/app/(app)/[workspace]/<route>/page.tsx` | Next.js convention |
| An authenticated route handler | `apps/web/app/api/<area>/route.ts` | Next.js convention |
| A new feature module | `features/<feature-name>/` | kebab-case folder |
| Feature-only API logic | `features/<feature>/api/<verb>-<noun>.ts` | imperative-noun |
| Feature-only component | `features/<feature>/components/<Name>.tsx` | PascalCase |
| Shared component (≥2 features use it) | `packages/ui/` or `apps/web/components/ui/` | PascalCase |
| A pure utility | `apps/web/lib/<area>/<name>.ts` | camelCase |
| A Zod schema | `features/<feature>/schemas/<name>.ts` | camelCase |
| A Supabase migration | `packages/db/migrations/<timestamp>_<description>.sql` | timestamped |
| RLS policies | Same migration as the table or `packages/db/migrations/<ts>_rls_<table>.sql` | timestamped |
| n8n workflow export | `n8n/<workflow-name>.json` | kebab-case |
| ADR | `docs/decisions/NNNN-<kebab-title>.md` | sequential |
| Ghost bug log | `memory/ghost-bugs.md` (append) | n/a |
| Handoff note | `memory/handoff.md` (overwrite or append depending on rule) | n/a |

## Forbidden

- **Importing from another feature's internals** (e.g., `features/billing/lib/foo` from `features/onboarding/`). Go through `features/billing/facade.ts` or extract the shared piece to `packages/`.
- **Cross-feature shared state outside `lib/`.** Global state for multiple features must live in a designated lib.
- **Files > 300 lines.** Split into a folder with named children.
- **Default exports.** Named exports only — prevents misuse and improves IDE rename refactors.
- **ORM objects returned from facades.** Return frozen DTOs (immutable, plain data objects) to keep callers from accidentally coupling to ORM lifecycle.
- **Editing generated files** (`*.generated.ts`, `supabase/types.ts`, `.next/`, `dist/`). They're regenerated by tooling.
- **Mixing dev and build in the same checkout.** Killing dev → `rm -rf .next dist` → build → relaunch dev. Per Sebastian's global rule.

## Naming patterns

| Thing | Pattern | Example |
|---|---|---|
| Component file | `PascalCase.tsx` | `BillingPlanCard.tsx` |
| Utility file | `camelCase.ts` | `formatCurrency.ts` |
| Server action | `camelCase.ts` (imperative-noun) | `createSubscription.ts` |
| Zod schema | `camelCase.ts` | `subscriptionSchema.ts` |
| Folder | `kebab-case/` | `billing-portal/` |
| Migration | `<unix-ts>_<snake_case>.sql` | `1715789324_create_subscriptions.sql` |
| ADR | `NNNN-kebab-title.md` | `0007-mermaid-c4-only.md` |
| Test | `<original-filename>.test.ts(x)` | `BillingPlanCard.test.tsx` |
| Branch | `<type>/<short>` | `feat/billing-portal` |
| Commit | `<type>(<scope>): <imperative>` | `feat(billing): add upgrade flow` |

---

**Last updated:** 2026-05-26
**Last verified:** 2026-05-26 — nightly audit (read-only review; no content changes)

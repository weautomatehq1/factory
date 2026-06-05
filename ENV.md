---
file: ENV.md
status: accepted
last_updated: 2026-05-26
owner: sebas
related_adrs: []
---

# Environment Variables

> Twelve-Factor Config (12factor.net/config). **Names only — NEVER values.** Pre-commit hook rejects any commit matching common secret patterns (`sk_`, `xoxb-`, JWT prefix `eyJ`, AWS key prefixes, etc.).
>
> The Factory repo (this) declares vars its tooling and its sibling repos need. Each per-client repo has its own ENV.md with client-specific keys.

## Schema

Every env var row has:
- `name` — SHOUT_SNAKE_CASE
- `purpose` — one line, plain English
- `source` — where the value comes from (1Password vault, dashboard, terraform output, per-deploy)
- `scope` — `build` | `runtime` | `server-only` | `client` (client requires `NEXT_PUBLIC_` prefix per Next.js convention)
- `required` — `yes` | `no` | `conditional:<env>`
- `rotation` — `never` | `<N>d` | `on-incident`
- `owner` — team / person responsible
- `consumed_by` — which repo(s) read it

## Storage

| Tier | Storage | Access |
|---|---|---|
| Development | `.env.local` per repo (gitignored) + Sebastian's password manager | Sebastian + Esme |
| Staging | Vercel environment vars (per-project) | Sebastian via Vercel dashboard |
| Production | Vercel + Hostinger env files + 1Password vault as source of truth | Sebastian only for production secrets |
| n8n credentials | n8n's built-in credential store (encrypted at rest) | weautomatehq.cloud admin |

## The Factory + sibling repos — required vars

### Authentication & APIs

| name | purpose | source | scope | required | rotation | owner | consumed_by |
|---|---|---|---|---|---|---|---|
| `ANTHROPIC_API_KEY` | Claude API access (synthesizer + workers) | console.anthropic.com | runtime, server-only | yes | 180d | sebas | IFleet, voice-discovery |
| `OPENAI_API_KEY` | OpenAI Codex worker (when active) | platform.openai.com | runtime, server-only | conditional:codex-enabled | 180d | sebas | IFleet |
| `RETELL_API_KEY` | Retell AI voice platform | dashboard.retellai.com | runtime, server-only | yes | 180d | sebas | voice-discovery |
| `RETELL_PHONE_NUMBER` | Provisioned phone for inbound/outbound | dashboard.retellai.com | runtime, server-only | yes | never | sebas | voice-discovery |
| `RETELL_WEBHOOK_SECRET` | HMAC for verifying Retell webhooks | dashboard.retellai.com | runtime, server-only | yes | on-incident | sebas | voice-discovery, n8n |

### Storage & Database

| name | purpose | source | scope | required | rotation | owner | consumed_by |
|---|---|---|---|---|---|---|---|
| `SUPABASE_URL` | Project endpoint (factory + per-client) | supabase.com dashboard | runtime | yes | never | sebas | voice-discovery, per-client |
| `SUPABASE_ANON_KEY` | Public-safe client key | supabase.com dashboard | runtime, client (with `NEXT_PUBLIC_` prefix) | yes | never | sebas | per-client web app |
| `SUPABASE_SERVICE_KEY` | Server-side admin key (bypasses RLS) | supabase.com dashboard | runtime, server-only | yes | 90d | sebas | server actions, IFleet schema worker |
| `DATABASE_URL` | Direct Postgres connection (migrations) | supabase.com dashboard | build + runtime, server-only | yes | when service-key rotates | sebas | migration tooling |

### Orchestration

| name | purpose | source | scope | required | rotation | owner | consumed_by |
|---|---|---|---|---|---|---|---|
| `N8N_API_KEY` | API access to weautomatehq.cloud | n8n self-hosted instance | runtime, server-only | yes | 90d | sebas | IFleet bootstrapper, voice-discovery synthesizer |
| `N8N_WEBHOOK_BASE` | Base URL for inbound webhooks | per-deploy config | runtime | yes | never | sebas | Retell → n8n |
| `GITHUB_TOKEN` | PAT for repo ops (create, commit, PR) | github.com/settings/tokens | runtime, server-only | yes | 90d | sebas | IFleet, voice-discovery synthesizer |
| `DISCORD_BOT_TOKEN` | Discord bot identity | discord.com/developers | runtime, server-only | yes | on-incident | sebas | discord-claude-bot |
| `DISCORD_CHANNEL_IFLEET` | Channel ID for IFleet briefs (1504120127791042631) | discord (channel right-click → copy ID) | runtime | yes | never | sebas | discord-claude-bot, n8n digest |
| `DISCORD_USER_LABEL` | Outgoing prefix (`Sebas` or `Esmel`) | per-machine config | runtime | yes | never | per-user | discord-claude-bot |

### Deploy & Hosting

| name | purpose | source | scope | required | rotation | owner | consumed_by |
|---|---|---|---|---|---|---|---|
| `VERCEL_TOKEN` | API access for deploy worker | vercel.com/account/tokens | runtime, server-only | yes | 90d | sebas | IFleet deploy worker |
| `VERCEL_PROJECT_ID` | Per-project ID (set per-client) | vercel.com project settings | runtime | yes | never | sebas | per-client deploy |
| `HOSTINGER_API_KEY` | VPS + DNS API access | hpanel.hostinger.com | runtime, server-only | yes | 90d | sebas | bootstrapper for DNS |
| `HOSTINGER_VPS_SSH_KEY` | SSH key for deploy SSH ops | ~/.ssh/factory_deploy_ed25519 (vault-backed) | runtime, server-only | yes | 180d | sebas | deploy worker |

### Observability

| name | purpose | source | scope | required | rotation | owner | consumed_by |
|---|---|---|---|---|---|---|---|
| `SENTRY_DSN` | Error reporting endpoint (per-project) | sentry.io project settings | runtime | yes | when project recreated | sebas | per-client app |
| `SENTRY_AUTH_TOKEN` | API access for n8n → GitHub bridge | sentry.io org settings | runtime, server-only | yes | 180d | sebas | n8n Sentry bridge |
| `POSTHOG_KEY` | Analytics project key | posthog.com project settings | runtime, client (`NEXT_PUBLIC_`) | yes | never | sebas | per-client app |
| `POSTHOG_HOST` | EU / US / self-hosted endpoint | posthog.com | runtime, client (`NEXT_PUBLIC_`) | yes | never | sebas | per-client app |

### Self-healing killswitches

| name | purpose | source | scope | required | rotation | owner | consumed_by |
|---|---|---|---|---|---|---|---|
| `SELF_HEAL_BUILD` | Tier 1 (build-time CI failure healing) — `on` / `off` | per-deploy / Discord command | runtime | yes | never | sebas | IFleet |
| `SELF_HEAL_DEPLOY` | Tier 2 (deploy-time smoke fail rollback) — `on` / `off` | per-deploy / Discord command | runtime | yes | never | sebas | IFleet deploy worker |
| `SELF_HEAL_RUNTIME` | Tier 3 (Sentry/PostHog runtime fix PRs) — `on` / `off` | per-deploy / Discord command | runtime | yes (default `off`) | never | sebas | IFleet runtime listener |
| `OMC_KILLSWITCH` | Global OMC orchestration off-switch | shell env | runtime | no | never | sebas | claude-code OMC layer |

### Billing — Stripe (per-client, when applicable)

| name | purpose | source | scope | required | rotation | owner | consumed_by |
|---|---|---|---|---|---|---|---|
| `STRIPE_SECRET_KEY` | Server-side Stripe API key | dashboard.stripe.com | runtime, server-only | conditional:billing-enabled | 90d | sebas | per-client billing flows |
| `STRIPE_PUBLISHABLE_KEY` | Client-safe Stripe key — injected as `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` at deploy time (stored in vault as `STRIPE_PUBLISHABLE_KEY`, `NEXT_PUBLIC_` prefix added by Next.js build) | dashboard.stripe.com | runtime, client (with `NEXT_PUBLIC_` prefix) | conditional:billing-enabled | never | sebas | per-client web app |
| `STRIPE_WEBHOOK_SECRET` | Signing secret for Stripe webhook events | dashboard.stripe.com → Webhooks | runtime, server-only | conditional:billing-enabled | on-incident | sebas | per-client billing flows |

### Signing — Documenso

| name | purpose | source | scope | required | rotation | owner | consumed_by |
|---|---|---|---|---|---|---|---|
| `DOCUMENSO_API_KEY` | API key for e-signing requests | sign.weautomatehq.cloud | runtime, server-only | conditional:signing-enabled | 90d | sebas | client onboarding flow |

### Cost controls

| name | purpose | source | scope | required | rotation | owner | consumed_by |
|---|---|---|---|---|---|---|---|
| `DAILY_RATE_CAP_USD` | Soft cap per project per day (default 10) | config | runtime | yes | never | sebas | IFleet cost tracker (PR #39) |
| `OVERNIGHT_MODE` | `on` = unlimited (ralph/autopilot), `off` = capped | per-session | runtime | yes (default `off`) | never | sebas | IFleet |

---

## Reserved prefixes

- `NEXT_PUBLIC_*` — Next.js convention; exposed to client JS bundle. Use ONLY for non-sensitive values.
- `SUPABASE_*` — Supabase identifiers
- `RETELL_*` — Retell-platform-specific
- `SELF_HEAL_*` — self-healing killswitches
- `OMC_*` — oh-my-claudecode orchestration layer

## Enforced patterns

A pre-commit hook rejects commits where any tracked `.md` / `.json` / source file contains a value matching:

- `sk_(test|live)_[a-zA-Z0-9]{20,}` — Stripe
- `xoxb-[a-zA-Z0-9-]+` — Slack
- `eyJ[a-zA-Z0-9_-]+\.[a-zA-Z0-9_-]+\.[a-zA-Z0-9_-]+` — JWT (with two dots)
- `AKIA[0-9A-Z]{16}` — AWS access key ID
- `ghp_[a-zA-Z0-9]{36}` — GitHub PAT
- `sentry_dsn_[a-zA-Z0-9_-]+` — Sentry DSN guard
- Generic high-entropy strings ≥ 32 chars in a context that doesn't look like prose

False positives in prose: surround with `<example>...</example>` tags in markdown to whitelist.

## Rotation calendar

> Note: `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, and `DOCUMENSO_API_KEY` appear in this calendar only when their feature is enabled (`required: conditional:billing-enabled` / `conditional:signing-enabled`). Skip those rows when the corresponding feature is disabled.

| Cadence | Items | Trigger |
|---|---|---|
| 90 days | `SUPABASE_SERVICE_KEY`, `GITHUB_TOKEN`, `VERCEL_TOKEN`, `HOSTINGER_API_KEY`, `N8N_API_KEY`, `STRIPE_SECRET_KEY`†, `DOCUMENSO_API_KEY`† | quarterly cron + ADR check |
| 180 days | `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `RETELL_API_KEY`, `SENTRY_AUTH_TOKEN`, `HOSTINGER_VPS_SSH_KEY` | half-yearly |
| Never | `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `DISCORD_CHANNEL_IFLEET`, `RETELL_PHONE_NUMBER`, `STRIPE_PUBLISHABLE_KEY`† | n/a |
| On incident | `RETELL_WEBHOOK_SECRET`, `DISCORD_BOT_TOKEN`, `STRIPE_WEBHOOK_SECRET`†, anything suspected leaked | as needed |

† Conditional — rotate only when billing/signing feature is enabled.

When rotating: update 1Password vault → update each deployment target → confirm with synthetic health-check → document the rotation in `CHANGELOG.md` under `Security`.

---

**Last updated:** 2026-06-05
**Last verified:** 2026-06-05 — nightly audit (rotation calendar conditional note added; closes AUDIT-factory-c856e7d6)

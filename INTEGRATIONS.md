---
file: INTEGRATIONS.md
status: accepted
last_updated: 2026-05-26
owner: sebas
related_adrs: [ADR-0006]
---

# Integrations Registry

> Format inspired by Backstage `catalog-info.yaml`. Each external service is a Resource or API entity. **Read-only for AI workers** — adding / removing / re-scoping an integration is a security boundary change and requires HITL approval. (See ADR — Add Integration HITL when written.)
>
> Per-service contract:
> - `kind` — Resource | API | Tool
> - `type` — payments | email | auth | storage | ai | observability | comms | voice | code | hosting | analytics | signing
> - `purpose` — one line
> - `auth_model` — api_key | oauth2 | jwt | hmac | mtls | none
> - `secret_refs` — env var names from `ENV.md` (NEVER values)
> - `scope` — which workspaces/users it touches
> - `rate_limits` — known limits
> - `retry_policy` — exponential / linear / none + max attempts
> - `status_page` — vendor's status URL
> - `docs` — vendor's API docs URL
> - `owner` — internal owner
> - `lifecycle` — production | experimental | deprecated
> - `depended_on_by` — internal components that depend on this
> - `last_audit` — date of most recent security audit

---

## Voice / Telephony

### Retell AI
- **kind:** API
- **type:** voice
- **purpose:** voice + telephony + STT + TTS + barge-in for the discovery interviewer
- **auth_model:** api_key + hmac (for webhooks)
- **secret_refs:** [`RETELL_API_KEY`, `RETELL_PHONE_NUMBER`, `RETELL_WEBHOOK_SECRET`]
- **scope:** WeAutomateHQ-wide; clients receive/place calls but don't have direct Retell access
- **rate_limits:** per Retell plan (TBD from Esme R1 research)
- **retry_policy:** exponential, max 3; on persistent fail, n8n escalates to Discord
- **status_page:** https://status.retellai.com ⚠️ [UNVERIFIED — confirm after account creation; update this line with verified date]
- **docs:** https://docs.retellai.com
- **owner:** sebas
- **lifecycle:** production (pending account creation D2)
- **depended_on_by:** voice-discovery
- **last_audit:** n/a (new integration)

---

## Storage & Database

### Supabase
- **kind:** Resource
- **type:** storage + auth + realtime
- **purpose:** Database (transcripts, coverage, per-client data), Auth (per-client SaaS), Realtime (dashboard live transcript), Storage (recordings if enabled)
- **auth_model:** jwt (anon + authenticated keys) + service-role key for server
- **secret_refs:** [`SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_KEY`, `DATABASE_URL`]
- **scope:** per-project Supabase instance (one for voice-discovery, one per client)
- **rate_limits:** per Supabase plan; pgvector queries can throttle on Free tier
- **retry_policy:** none at SDK level (Postgres errors surface immediately); n8n retries with backoff
- **status_page:** https://status.supabase.com
- **docs:** https://supabase.com/docs
- **owner:** sebas
- **lifecycle:** production
- **depended_on_by:** voice-discovery, IFleet schema worker, every per-client repo
- **last_audit:** n/a

---

## Code & Version Control

### GitHub (weautomatehq1 org)
- **kind:** API
- **type:** code + Issues queue
- **purpose:** Repo hosting + Issues (the unit of work, per ADR-0003) + branch protection + Actions CI + Pages
- **auth_model:** PAT (fine-grained per env) + GitHub App (planned)
- **secret_refs:** [`GITHUB_TOKEN`]
- **scope:** weautomatehq1 org; per-client repos created under same org
- **rate_limits:** 5000 req/hour for PAT; 15000 for GitHub App
- **retry_policy:** exponential, max 5; respect `Retry-After` header
- **status_page:** https://www.githubstatus.com
- **docs:** https://docs.github.com/rest
- **owner:** sebas
- **lifecycle:** production
- **depended_on_by:** IFleet (decomposer, queue bridge, every worker), voice-discovery synthesizer, this repo
- **last_audit:** 2026-05-12 (IFleet branch protection ruleset 16316354 confirmed)

---

## AI Models

### Anthropic Claude (API + Code)
- **kind:** API
- **type:** ai
- **purpose:** Synthesizer brain (Opus), worker pipeline (Sonnet default, Opus on complexity:high per IFleet model routing decision 2026-05-12; see STAKEHOLDERS.md decision log)
- **auth_model:** api_key
- **secret_refs:** [`ANTHROPIC_API_KEY`]
- **scope:** all repos
- **rate_limits:** 5-hour rate window (RPM + tokens); long briefs on Opus can hit cap silently
- **retry_policy:** exponential + provider-rotation fallback; downshift Opus→Sonnet on rate-limit
- **status_page:** https://status.anthropic.com
- **docs:** https://docs.anthropic.com
- **owner:** sebas
- **lifecycle:** production
- **depended_on_by:** IFleet workers, voice-discovery synthesizer, this Claude session
- **last_audit:** 2026-05-13 (Phase A smoke green confirmed model routing)

### OpenAI Codex (Pro)
- **kind:** API
- **type:** ai
- **purpose:** Codex worker for cross-provider review (planned per [[ifleet-repo]] memory)
- **auth_model:** api_key
- **secret_refs:** [`OPENAI_API_KEY`]
- **scope:** IFleet only
- **rate_limits:** per Codex Pro plan
- **retry_policy:** exponential, max 3
- **status_page:** https://status.openai.com
- **docs:** https://platform.openai.com/docs
- **owner:** sebas
- **lifecycle:** experimental (Codex Pro to be purchased — see [[ifleet-repo]])
- **depended_on_by:** IFleet (planned)
- **last_audit:** n/a

---

## Automation Platform

### n8n (self-hosted)
- **kind:** Resource + API
- **type:** orchestration
- **purpose:** Webhook receiving, multi-step workflows, OAuth handling for client integrations, audit-trail UI
- **auth_model:** api_key (for n8n API) + per-credential auth stored in n8n's credential store
- **secret_refs:** [`N8N_API_KEY`, `N8N_WEBHOOK_BASE`]
- **scope:** WeAutomateHQ-wide; visible to clients (agency value)
- **rate_limits:** self-hosted, limited by VPS resources
- **retry_policy:** native n8n retry config per node
- **status_page:** https://status.weautomatehq.cloud ⚠️ [UNVERIFIED — page may not exist yet; during incidents use host monitoring directly]
- **docs:** https://docs.n8n.io
- **owner:** sebas
- **lifecycle:** production
- **depended_on_by:** voice-discovery (Retell→Supabase), IFleet (Sentry bridge, bootstrapper, deploy worker), per-client workflows
- **last_audit:** 2026-05-13 (Sentry→GitHub bridge workflow deployed; webhook URL wire pending)

---

## Communications

### Discord
- **kind:** API + Bot
- **type:** comms
- **purpose:** Operator channel for briefs, daily digest (M-018), bot-driven status pings
- **auth_model:** bot token + per-channel permissions
- **secret_refs:** [`DISCORD_BOT_TOKEN`, `DISCORD_CHANNEL_IFLEET`]
- **scope:** WeAutomateHQ Discord server
- **rate_limits:** 50 requests/sec global; per-channel 5/5s
- **retry_policy:** native discord.js handles rate-limit backoff
- **status_page:** https://discordstatus.com
- **docs:** https://discord.com/developers/docs
- **owner:** sebas
- **lifecycle:** production
- **depended_on_by:** discord-claude-bot (`~/dev/coordination/discord-claude-bot/`)
- **last_audit:** 2026-05-13 (Discord MCP env reference per `[[reference_discord_mcp_env]]`)

---

## Hosting

### Vercel
- **kind:** Resource + API
- **type:** hosting
- **purpose:** Preview deploys per PR (M-013), production hosting for per-client web apps
- **auth_model:** api_token
- **secret_refs:** [`VERCEL_TOKEN`, `VERCEL_PROJECT_ID`]
- **scope:** per-project (one Vercel project per client repo)
- **rate_limits:** per Vercel plan
- **retry_policy:** native; CI surfaces failures, IFleet deploy worker decides retry
- **status_page:** https://www.vercel-status.com
- **docs:** https://vercel.com/docs
- **owner:** sebas
- **lifecycle:** production
- **depended_on_by:** IFleet deploy worker, per-client web apps
- **last_audit:** n/a

### Hostinger
- **kind:** Resource + API
- **type:** hosting + dns
- **purpose:** VPS for non-Vercel surfaces (n8n, Documenso, etc.), DNS records, SSL
- **auth_model:** api_key + SSH key
- **secret_refs:** [`HOSTINGER_API_KEY`, `HOSTINGER_VPS_SSH_KEY`]
- **scope:** WeAutomateHQ infra
- **rate_limits:** per Hostinger plan
- **retry_policy:** ssh ops retry once, then escalate to Discord
- **status_page:** https://www.hostinger.com/status ⚠️ [UNVERIFIED — confirm URL works before first incident]
- **docs:** https://developers.hostinger.com
- **owner:** sebas
- **lifecycle:** production
- **depended_on_by:** n8n instance, Documenso instance, bootstrapper for DNS

---

## Observability

### Sentry
- **kind:** API
- **type:** observability
- **purpose:** Error tracking + breadcrumb trail; feeds Tier 3 self-healing trigger via n8n bridge
- **auth_model:** DSN (writes) + auth_token (reads + API)
- **secret_refs:** [`SENTRY_DSN`, `SENTRY_AUTH_TOKEN`]
- **scope:** per-project (one Sentry project per client + one for The Factory infra)
- **rate_limits:** per plan
- **retry_policy:** Sentry SDK handles natively
- **status_page:** https://status.sentry.io
- **docs:** https://docs.sentry.io
- **owner:** sebas
- **lifecycle:** production
- **depended_on_by:** all per-client apps, IFleet self-healing Tier 1+2
- **last_audit:** 2026-05-13 (Sentry MCP + n8n bridge deployed per `[[project-status-20260513]]`)

### PostHog
- **kind:** API
- **type:** analytics
- **purpose:** Product analytics, feature flags, A/B; feeds Tier 3 anomaly detection
- **auth_model:** api_key
- **secret_refs:** [`POSTHOG_KEY`, `POSTHOG_HOST`]
- **scope:** per-project per client
- **rate_limits:** per plan
- **retry_policy:** PostHog SDK handles
- **status_page:** https://status.posthog.com
- **docs:** https://posthog.com/docs
- **owner:** sebas
- **lifecycle:** production
- **depended_on_by:** all per-client apps (when analytics scope agreed with client)

---

## Billing (per-client, when applicable)

### Stripe
- **kind:** API
- **type:** payments
- **purpose:** Per-client SaaS billing (subscriptions, one-time, usage-based)
- **auth_model:** api_key (server) + publishable_key (client) + webhook signing secret
- **secret_refs:** [`STRIPE_SECRET_KEY`, `STRIPE_PUBLISHABLE_KEY`, `STRIPE_WEBHOOK_SECRET`] (per client)
- **scope:** per-client account
- **rate_limits:** 100 req/s burst, 25 req/s sustained
- **retry_policy:** exponential with idempotency keys
- **status_page:** https://status.stripe.com
- **docs:** https://stripe.com/docs/api
- **owner:** sebas
- **lifecycle:** production
- **depended_on_by:** per-client billing flows
- **last_audit:** n/a

---

## Signing

### Documenso (self-hosted at sign.weautomatehq.cloud)
- **kind:** Resource + API
- **type:** signing
- **purpose:** E-signing for client contracts / SOWs / data processing agreements
- **auth_model:** api_key
- **secret_refs:** [`DOCUMENSO_API_KEY`]
- **scope:** WeAutomateHQ-wide
- **rate_limits:** self-hosted, VPS-limited
- **retry_policy:** native + manual retry on UI
- **status_page:** none (self-hosted)
- **docs:** https://documenso.com/docs
- **owner:** sebas
- **lifecycle:** production
- **depended_on_by:** client onboarding flow (when SOW signing needed)

---

## Tools (read-only, no auth required)

### context7 MCP
- **kind:** Tool
- **type:** docs
- **purpose:** Live library/framework docs lookup (Next.js, Supabase, Stripe, etc.)
- **auth_model:** none
- **lifecycle:** production

### Lightpanda MCP
- **kind:** Tool
- **type:** web
- **purpose:** Web research / page content extraction
- **auth_model:** none
- **lifecycle:** production

### SearXNG (self-hosted MCP)
- **kind:** Tool
- **type:** search
- **purpose:** Search across the web via self-hosted instance
- **auth_model:** none (localhost binding)
- **lifecycle:** production

### GitHub MCP
- **kind:** Tool
- **type:** code
- **purpose:** GitHub API access for Claude Code
- **auth_model:** uses `GITHUB_TOKEN`
- **lifecycle:** production

---

## Worker rules (agent contract)

- This file is **READ-ONLY** for AI workers. They may read it to discover which service handles a capability, look up rate limits before retrying, or check status page URLs before declaring outage.
- **Adding** a new integration = HITL (see `STAKEHOLDERS.md` Approver column).
- **Removing** an integration = HITL.
- **Changing** auth_model or secret_refs = HITL + security audit.
- **Updating** rate limits / docs URLs / lifecycle status = OK to PR.

---

**Last updated:** 2026-06-06
**Last verified:** 2026-06-06 — nightly audit (footer date refreshed; closes AUDIT-factory-20260606-09)

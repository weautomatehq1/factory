---
file: NON_GOALS.md
status: accepted
last_updated: 2026-05-26
owner: sebas
related_adrs: [ADR-0004]
---

# Non-Goals

> Two categories, both load-bearing for the worker fleet's safety:
>
> - **Non-Goals** — could reasonably be goals, explicitly deferred this release. Revisit later.
> - **No-Gos** — never, walk away. (Shape Up convention.)
>
> If a worker proposes work that hits either, the proposal is rejected. No further review needed.

## Why this file exists

The worker fleet will build whatever is plausibly in scope. Without explicit fences, it will gold-plate, scope-creep, and "improve" things we deliberately left out. This file is the fence.

This file is also a safety rail for self-healing — Tier 3 runtime patches that touch No-Gos auto-reject regardless of any other gate.

## Goals (for context — the actual roadmap is `ROADMAP.md`)

The Factory at full operation:
1. Voice AI interviews client → produces 17 spec files within 30 min of call ending
2. Operator (Sebastian + Esme) review spec PR within 2 hours, merge
3. IFleet decomposes spec → emits dependency-ordered issues
4. Workers build the platform, opening PRs as they go
5. Operator approves PRs daily (~30 min/day)
6. Self-healing handles >70% of failures without operator intervention
7. Platform launches behind 7 human gates (spec, design, copy, Stripe, DNS, compliance, go-live)
8. Post-launch: Tier 3 self-healing catches and fixes runtime issues

## Non-Goals (this release — 5-day sprint ending 2026-05-20)

### NG-1. Tier 3 runtime self-healing
- **Reasonable as a goal?** Yes.
- **Why excluded:** No production data to tune gates against. Auto-merge thresholds (risk score, screenshot diff sensitivity) need real Sentry/PostHog signal to calibrate.
- **Boundary:** Tier 1 (build) + Tier 2 (deploy) only this sprint. Tier 3 deferred to M-017, opens after client #1 has 2 weeks of clean production.
- **Source:** ADR-0005

### NG-2. Per-client custom branding for The Factory operator dashboard
- **Reasonable as a goal?** Yes (clients might want their own logo on the live-call view).
- **Why excluded:** The dashboard is internal to WeAutomateHQ. Clients see staging URLs of their own product, not the dashboard.
- **Boundary:** Single WeAutomateHQ-branded dashboard. No theming.

### NG-3. Multi-language voice interviews
- **Reasonable as a goal?** Yes — future clients will be Spanish-speaking.
- **Why excluded:** Adds prompt-tuning cycles per language. Validate the pattern in English first.
- **Boundary:** English only this sprint. Spanish deferred to M-023.

### NG-4. Real-time spec preview during interview
- **Reasonable as a goal?** Yes — operator could see spec sections being filled live.
- **Why excluded:** Coverage heatmap (M-007) gives 80% of the value. Full preview is post-call.
- **Boundary:** Heatmap + transcript stream only. Synthesizer runs after call ends.

### NG-5. Sophisticated cost-attribution per client
- **Reasonable as a goal?** Yes — knowing exact cost per client matters for pricing.
- **Why excluded:** Day-1 has 1 client. Aggregate cost in `.omc/costs.json` is enough until N≥3.
- **Boundary:** `costs.json` total only. Per-client breakdown deferred to M-019.

### NG-6. Optimizing IFleet worker model selection
- **Reasonable as a goal?** Yes — cost vs quality is real.
- **Why excluded:** Phase B model decision (sonnet default, opus on complexity:high) is locked. Re-tuning mid-sprint = sprint death.
- **Boundary:** Default routing.json. Re-evaluate after 30 days of real data.

### NG-7. Comprehensive test coverage for new IFleet gaps
- **Reasonable as a goal?** Yes — debt grows otherwise.
- **Why excluded:** Smoke tests get us to operational. Full coverage in next cycle.
- **Boundary:** Smoke test per gap, no unit-test sweep this sprint. Trade-off documented; debt is intentional.

### NG-8. Documentation site (Mintlify-published spec files)
- **Reasonable as a goal?** Yes — clients could read ARCHITECTURE.md in a nice UI.
- **Why excluded:** GitHub renders markdown fine. Mintlify is a polish add.
- **Boundary:** Plain GitHub render only. Mintlify deferred.

---

## No-Gos (never, not "not yet")

These are walking-away decisions. If a feature/scope hits a No-Go, we don't do it — full stop. Adding to this list requires opening an ADR; removing requires opening an ADR.

### NoGo-1. The Factory is NOT a productized dev tool
- We will NOT compete with Lovable / Bolt / Cursor / v0 / Replit Agent / Devin.
- The Factory is **internal agency leverage**. The product we sell is the SaaS we build for the client, not the system that built it.
- Reason: distinct market, distinct customer (developers vs business owners), distinct support requirements. We don't have the bandwidth or interest for a dev-tool customer base.
- Documented: ADR-0004

### NoGo-2. NOT supporting non-SaaS deliverables
- Mobile apps (native iOS/Android) — out of competence zone
- Embedded firmware — out of stack
- ML training pipelines — out of stack
- Hardware integrations — out of stack
- Browser extensions — out of stack
- Native desktop apps — out of stack
- Reason: each adds an entirely new stack to support. We've narrowed to SaaS web + adjacent integrations.

### NoGo-3. NOT auto-merging changes to Protected Paths
- Auth code (`lib/auth/`, OAuth flows, JWT handling) — always human-merge
- Billing code (Stripe handlers, subscription state, refunds) — always human-merge
- PII handlers (user profile writes, audit logs) — always human-merge
- Database migrations — always human-merge
- ENV.md / SECURITY.md / `.well-known/security.txt` — always human-merge
- `/infrastructure/**` — always human-merge
- Reason: blast radius of an auto-merge mistake here is unacceptable. No risk score is low enough.

### NoGo-4. NOT replacing the client's CRM of record
- If client uses HubSpot, we integrate WITH HubSpot. We don't migrate them off.
- Same for Salesforce, Pipedrive, etc.
- Reason: CRM migrations are 10x bigger projects than SaaS builds. Not our scope.

### NoGo-5. NOT shipping without operator approval at the 7 gates
- Spec PR approval — always human
- Design system approval — always human
- Copy approval (public-facing pages) — always human
- Stripe live key paste — always human
- DNS cutover — always human
- Compliance review (if regulated) — always human
- Go-live approval — always human
- Reason: these are commitments to the client. No autonomy here.

### NoGo-6. NOT building for clients who can't be voice-interviewed
- Deaf clients without sign-language workflow — different product
- Text-only-preference clients — different product
- Clients who refuse a recorded call — different product
- Reason: voice interview is the foundational primitive of The Factory. Other intake forms = different system entirely.
- (Future: a text-based intake mode is plausible but is M-026+, not this product line.)

### NoGo-7. NOT fabricating INTAKE.md content
- If we don't have a real client interview, INTAKE stays `status: skeleton`.
- We never write fake quotes, fake observed facts, fake emotional signals.
- Reason: AI workers downstream cite INTAKE as truth. Fake INTAKE = fake spec = fake product.

### NoGo-8. NOT skipping the ADR for major decisions
- Architectural choices, integration additions, security trade-offs all get an ADR.
- Reason: institutional memory; worker fleet won't try to "fix" decisions it doesn't know about.

### NoGo-9. NOT running `pnpm dev` and `pnpm build` on the same checkout
- Per Sebastian's global rule: corrupts `.next/`, causes React #418, blank screens.
- Reason: documented production incident pattern.

### NoGo-10. NOT committing secrets
- No `.env`, no API keys, no service account credentials, no Supabase service role keys.
- Pre-commit hook enforced.
- Reason: public repo + one leak = full rotation across all integrations.

---

**Last updated:** 2026-05-26
**Last verified:** 2026-05-26 — nightly audit (read-only review; no content changes)

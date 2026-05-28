---
file: SECURITY.md
status: accepted
last_updated: 2026-05-21
owner: sebas
related_adrs: [ADR-0009]
---

# Security

> ⚠️ **WARNING — enforcement controls newly wired (2026-05-21).** Until 2026-05-21, the pre-commit hook, CODEOWNERS file, and Protected-Paths GitHub Action described in this document existed in prose only — `.git/hooks/`, `.github/`, and `.claude/` were empty. The first PR (`fix(audit): close AUDIT-factory-b6ff2b68`) creates them: `.husky/pre-commit`, `.github/CODEOWNERS`, `.github/workflows/protected-paths.yml`, and `.claude/settings.json`. Treat the first real PR that touches a Protected Path as a live verification of the gate — confirm the workflow runs, the label is applied, and merge is blocked without the required approval. If any link in the chain misfires, treat it as a CRITICAL security incident and pause auto-merge across the fleet.

> STRIDE threat model + OWASP Top 10 (2021) mapping + OWASP ASVS Level 2 default + RFC 9116 security.txt + four-tier data classification with NIST CIA mapping. The "Protected Paths" section is the auto-merge gate that the worker fleet honors absolutely.

## Reporting a Vulnerability (security.txt-compatible)

```
Contact: mailto:security@weautomatehq.com
Encryption: https://weautomatehq.com/pgp-key.txt (TBD)
Preferred-Languages: en, es
Policy: https://weautomatehq.com/security-policy (TBD)
Expires: 2027-05-15T00:00:00Z
```

**To mirror into** `/.well-known/security.txt` verbatim once the marketing site is live. Keep `Expires` rolling — update annually.

---

## Threat Model (STRIDE)

| Threat | Property violated | Examples in The Factory | Controls |
|---|---|---|---|
| **Spoofing** | Authenticity | Fake Retell webhook posing as a real call; spoofed Discord message; PR opened by impersonated user | HMAC on Retell webhooks (`RETELL_WEBHOOK_SECRET`); GitHub signed commits; Discord bot token verification; OAuth on all client integrations |
| **Tampering** | Integrity | Modified spec PR mid-review; altered ADR with `status: accepted`; database row tampering | Branch protection on `main`; pre-commit hook rejects edits to accepted ADRs; Supabase RLS; audit log; signed commits |
| **Repudiation** | Non-repudiability | "I didn't approve that PR"; "I didn't say that in the interview" | Immutable Supabase audit log; INTAKE.md verbatim transcript; PR review history retained; ADR decision log |
| **Information Disclosure** | Confidentiality | Leaked API key in commit; PII in logs; over-exposed Supabase RLS | Pre-commit secret scanning; structured logging with redaction; RLS-by-default on Supabase; Protected Paths enforcement |
| **Denial of Service** | Availability | Flood on Retell webhook endpoint; runaway worker loop; n8n workflow infinite loop | Rate limiting on webhooks; cost ceiling (`DAILY_RATE_CAP_USD`); 3-consecutive-failure rule stops loops; killswitches |
| **Elevation of Privilege** | Authorization | Cross-tenant data access; worker bypassing reviewer; secret service-key in client bundle | Supabase RLS per workspace; api-guard middleware on every server action; service keys are `server-only` scope; reviewer pass is non-skippable |

## Data Classification

Four-tier industry pattern. Each tier maps to NIST SP 800-60 CIA impact for audit traceability.

| Tier | Examples (in The Factory + client repos) | Storage rules | Retention | NIST C/I/A (low/mod/high) |
|---|---|---|---|---|
| **Public** | Marketing copy, public spec files in spec-template, README | Any storage; CDN OK | Indefinite | L/L/L |
| **Internal** | Runbooks, IFleet operational metrics, this repo's content, internal Discord briefs | Private repos, SSO-only, no public CDN | 5y rolling | M/M/M |
| **Confidential** | Client interview transcripts, customer PII in client SaaS, per-client business data, Sebastian's auto-memory | Encrypted at rest (Supabase column encryption where applicable), RLS-enforced, audit-logged on access | Per contract (default 7y) | H/H/M |
| **Restricted** | Secrets, payment data, OAuth tokens, signing keys, health data | Secret manager only (1Password / Supabase vault / Vercel env), never logged, never on disk in plain text | Rotation per ENV.md schedule | H/H/H |

**Promotion rule:** when in doubt, classify higher. Restricted always overrides any conflicting access pattern.

---

## Protected Paths (the auto-merge gate)

Changes touching these paths **REQUIRE human review** — IFleet workers MAY NOT auto-merge, even on green CI + low risk score + outside business hours + all other gates passing.

### Glob list (enforced by CODEOWNERS + auto-merge gate)

```
# Auth and authorization
**/lib/auth/**
**/api/auth/**
**/middleware/auth*.ts
**/middleware/api-guard*.ts
**/(auth)/**

# Billing and payments
**/lib/billing/**
**/api/billing/**
**/api/webhooks/stripe/**
**/features/billing/**

# PII handlers
**/lib/pii/**
**/lib/audit/**
**/api/user/**
**/api/profile/**

# Database
**/packages/db/migrations/**
**/supabase/migrations/**
**/db/seed/**

# Infrastructure
**/infrastructure/**
**/terraform/**
**/.github/workflows/**
**/Dockerfile
**/docker-compose*.yml

# Security and ops files
SECURITY.md
ENV.md
INTEGRATIONS.md
.well-known/security.txt
.well-known/**

# AGENTS contract
AGENTS.md
CLAUDE.md
**/AGENTS.md
**/CLAUDE.md

# Spec files (specific subset that defines invariants)
ARCHITECTURE.md
NON_GOALS.md
STAKEHOLDERS.md
docs/decisions/*.md  # individual ADRs are append-only — modifying accepted ones is gate-blocked
```

### Rationale per group

- **Auth / billing / PII**: blast radius of mistake = customer harm. Always human.
- **Database**: migrations are irreversible in practice. Human signs off on schema changes.
- **Infrastructure**: changes here affect every deployed surface. Human.
- **Security / ops files**: loosening these weakens every safety rail.
- **AGENTS contract**: this is what workers READ; allowing workers to write it = unsafe self-modification.
- **Spec invariants**: ARCHITECTURE / NON_GOALS / STAKEHOLDERS are commitments; ADRs are decided.

### How the gate works (implementation contract)

- IFleet deploy worker / runtime self-heal worker checks the PR's changed-files list against this glob set BEFORE attempting auto-merge.
- Match = block auto-merge. Post Discord ping to operator: "PR #N touches Protected Path: <path>. Manual review required."
- No-match = continue to other auto-merge gates (CI, screenshot diff, risk score, etc.).
- Gate is implemented as a GitHub Action that runs on every PR; the action also adds a `protected-path-touched` label for visibility.

---

## OWASP Top 10 (2021) Mapping Checklist

| # | Category | Our control | Status |
|---|---|---|---|
| A01 | Broken Access Control | RLS on every workspace-scoped Supabase table; `withAuth()` / api-guard wrapper on every protected route | required for every client repo |
| A02 | Cryptographic Failures | TLS everywhere; secrets in vault; no homemade crypto; bcrypt/argon2 for any password storage | required |
| A03 | Injection | Zod at all API boundaries; parameterized Supabase queries only; no raw SQL string concat | required |
| A04 | Insecure Design | Threat model reviewed per feature (this file); STRIDE applied to new architecture | required |
| A05 | Security Misconfiguration | Security headers (CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy); no defaults left enabled | required for every client repo |
| A06 | Vulnerable & Outdated Components | Dependabot enabled; weekly audit run; lockfiles committed | required |
| A07 | Identification & Auth Failures | Supabase Auth; MFA available; rate-limit on login; session expiry policy | required for any client SaaS with users |
| A08 | Software & Data Integrity | Signed commits (planned); lockfiles committed; pre-commit hook + branch protection | required |
| A09 | Logging & Monitoring Failures | Sentry on every per-client app; structured logs to stdout; audit trail in Supabase for sensitive ops | required |
| A10 | Server-Side Request Forgery | Outbound allowlist for worker fetches; no user-supplied URLs hit external services without validation | required |

## ASVS Target

- **Default for client SaaS:** Level 2 (apps handling sensitive data — our default tier per OWASP guidance)
- **Restricted-tier surfaces (billing, auth):** Level 3
- **Marketing site, public docs, README, status pages:** Level 1

The Factory repo itself (markdown only) is **Level 1** — no code surface to attack.

---

## Killswitches (operator-controlled)

- `SELF_HEAL_BUILD=off` — disables Tier 1 (build healing) instantly
- `SELF_HEAL_DEPLOY=off` — disables Tier 2 (deploy rollback) instantly
- `SELF_HEAL_RUNTIME=off` — disables Tier 3 (runtime fix PRs) instantly (default `off` until M-017)
- `OMC_KILLSWITCH=on` — disables all oh-my-claudecode orchestration (the global escape hatch)
- KPI regression detected → **automatic** pause of all self-healing for that project until human clears

Toggling: SSH to relevant deployment, update env, restart. Discord command bridge planned (M-019).

---

## Security review cadence

| Item | Cadence | Owner |
|---|---|---|
| Re-read this file | Per sprint | Sebastian |
| Rotate `Expires` date in security.txt | Yearly | Sebastian |
| Audit Protected Paths glob coverage | Quarterly + on any new Protected category | Sebastian |
| Penetration test (per-client SaaS) | Pre-launch + yearly | TBD vendor |
| Dependency audit | Weekly (Dependabot auto-PRs) | IFleet |
| Secret rotation | Per ENV.md schedule | Sebastian |

---

**Last updated:** 2026-05-21
**Last verified:** 2026-05-21 — Sebastian

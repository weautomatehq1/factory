---
adr: "0009"
title: "Four-tier data classification with NIST C/I/A mapping"
status: accepted
date: 2026-05-15
deciders: [sebas]
consulted: [claude-orchestrator, security-research]
informed: [ifleet, voice-discovery, spec-template]
tags: [security, compliance]
supersedes: null
---

# ADR-0009: Four-tier data classification with NIST C/I/A mapping

## Context and Problem Statement

SECURITY.md needs a data classification scheme. Two competing approaches: the four-tier industry pattern (Public / Internal / Confidential / Restricted) is what most companies use day-to-day; NIST SP 800-60 classifies by C/I/A impact dimension (Confidentiality × Integrity × Availability, each Low/Mod/High), which is the audit-aligned standard.

## Decision Drivers

- Day-to-day usability for the operator (Sebastian/Esme) — needs to be quickly classifiable in a sentence
- Auditability for clients in regulated industries (HIPAA, GDPR, SOC2)
- AI workers need to map data to handling rules quickly (storage, retention, encryption)
- Compatibility with industry vendor expectations (Palo Alto, Fortra, Atlan all use four-tier)

## Considered Options

1. **Four-tier only** — Public / Internal / Confidential / Restricted; ignore NIST
2. **NIST C/I/A only** — 9 categories (3×3); audit-aligned
3. **Four-tier + NIST footnote** — primary classification is four-tier; each tier maps to NIST C/I/A triple for audit traceability

## Decision Outcome

Chosen: **Option 3 — four-tier primary, NIST C/I/A footnote per tier.**

Mapping:
- **Public** → L/L/L (Low confidentiality, Low integrity, Low availability impact if breached)
- **Internal** → M/M/M
- **Confidential** → H/H/M
- **Restricted** → H/H/H

Promotion rule: when in doubt, classify higher. Restricted overrides any conflicting access pattern.

### Why not the others?
- **Option 1 (four-tier only):** loses NIST traceability — auditors in regulated client work would have to retro-fit a mapping.
- **Option 2 (NIST only):** 9 categories is too many for daily use. Operator can't quickly say "this is C/H I/M A/L" without thinking.

## Consequences

- **Good:**
  - Daily-use classification is simple (one of four labels)
  - Audit-aligned (NIST mapping in footnote)
  - Industry-vendor-aligned (Palo Alto, Fortra, Atlan, NIST 800-60 all in scope)
  - AI workers map "Confidential" → "encrypted at rest, RLS, audit logged" without ambiguity
- **Bad:**
  - Two-layer scheme adds slight complexity. Acceptable.
- **Neutral:**
  - For non-regulated clients, NIST footnote is informational only

## Confirmation

- Verify: SECURITY.md "Data Classification" section uses both columns
- Verify: any new piece of data introduced is classified within 24 hours of being added
- Audit: at 90-day mark, sample 5 data items and verify their classification still holds

## Related

- Sources: NIST SP 800-60 (https://nvlpubs.nist.gov/nistpubs/legacy/sp/nistspecialpublication800-60v1r1.pdf); Palo Alto data-classification cyberpedia
- Related: SECURITY.md "Data Classification"
- Related risks: R-003 (secret committed), R-015 (INTAKE fabrication — affects Confidential tier)

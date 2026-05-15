---
file: INTAKE.md
status: skeleton
populated_when: first real client voice interview completes AND transcript is persisted to Supabase
last_updated: 2026-05-15
owner: sebas
discipline: |
  - NEVER fabricate content here
  - Verbatim transcript only; synthesis lives in other files
  - Every quote MUST have a timestamp
  - Status stays `skeleton` until real data; agents read status before treating content as canonical
---

# Intake

> **Status: skeleton.** This file is the RAW EVIDENCE bucket. It is intentionally empty until a real client interview is captured. We do not fabricate quotes, observed facts, or emotional signals — `NoGo-7` in `NON_GOALS.md` is explicit.
>
> When populated, this file becomes the single source of truth for everything downstream. The Synthesizer (M-010) reads this to generate the other 16 spec files. If it isn't here, it didn't happen.

## Per-interview template

When the voice interviewer completes a call, the Synthesizer (M-010) produces an INTAKE.md following this exact structure:

```markdown
# INTAKE — {Client Name} — {YYYY-MM-DD}

## 0. Meta
- Interviewer: voice-discovery agent v{N}.{M} (or human if escalated)
- Duration: HH:MM
- Recording: <Supabase Storage URL>
- Participants: <names + roles>
- Stage: discovery | follow-up | scoping
- Coverage at end: <0.0-1.0 per required section>

## 1. Verbatim transcript
Full speaker-labeled transcript. NO edits. Timestamps every ~30s.
**Speaker:** "exact words..."
[12:14] Client: "exact quote..."
[12:32] Agent: "exact response..."

## 2. Direct quotes (pulled)
Quotes carrying weight — pain points, goals, money, names, vendor names, workarounds, emotional moments.
One bullet per quote, with timestamp + which spec section it informs.
- [12:14] "We re-key every appointment from the form into our CRM. It's nuts." → feeds ARCHITECTURE (data flow), KPI (time-saved metric)
- [18:02] "If it broke for a day we'd lose maybe two thousand dollars." → feeds RISKS (R-NEW: outage cost), KPI (uptime target)

## 3. Observed facts (not opinions)
What they DO today, not what they WOULD do. Past tense, concrete.
- Currently uses HubSpot Free + 2 Zapier flows (verified by them describing the screen)
- Last booking error happened "two weeks ago," cost ~4 hours

## 4. Emotional signals
Energy, sighs, accentuation, hedges. Per Bob Moesta: "Listen for the energy."
- [09:30] Long sigh when describing onboarding step → emotional weight on this pain
- [22:11] Voice raised when mentioning competitor X → anti-pattern, feeds NON_GOALS

## 5. Commitments offered
Per Mom Test: time / reputation / money. Anything they actually GAVE.
- Agreed to a 30-min follow-up Tuesday
- Will intro us to operations lead
- Verbal commit: "I'd pay $500/mo for this if it worked"

## 6. Open loops / things to verify
Questions we couldn't fully answer in-call. Carry into next conversation.
- What's their actual budget ceiling?
- Is the 17-field intake form fixed or negotiable?
- Who else needs to approve before they sign SOW?

## 7. Domain vocabulary captured
New terms heard for the first time → feed UBIQUITOUS_LANGUAGE.md
- "matter" → confirmed [08:14] means a legal work item
- "intake form" → 17-field form, lands in HubSpot

## 8. Coverage map snapshot
| Section | Filled? | Confidence | Source quotes |
|---|---|---|---|
| business_goal | yes | 0.9 | [00:32], [02:18] |
| target_user | yes | 0.85 | [04:12] |
| jobs_to_be_done | yes | 0.8 | [09:30], [12:14] |
| current_workflow | yes | 0.95 | [05:00 - 11:00] |
| pain_points | yes | 0.9 | [12:14], [18:02], [22:11] |
| success_metrics | partial | 0.6 | [25:00] — vague, needs follow-up |
| non_goals | yes | 0.7 | [29:14] explicit no |
| integrations | yes | 0.85 | [15:33] |
| data_model_hints | yes | 0.75 | [16:40] terms enumerated |
| compliance_needs | low | 0.4 | not deeply probed |
| brand_voice | partial | 0.5 | [31:00] mention |
| budget_timeline | yes | 0.8 | commitment [38:00] |
```

---

## Forbidden in this file

Per NON_GOALS NoGo-7:
- No fabricated quotes
- No synthesized claims ("the client probably wants...")
- No feature ideas authored by us — those go in PITCH (created post-call)
- No domain definitions — those go in UBIQUITOUS_LANGUAGE.md
- No scope decisions — those go in NON_GOALS.md
- No paraphrased text without timestamp

If a fact is not a direct quote, an observed action, or an emotional marker, it doesn't belong here.

---

## How downstream files cite INTAKE

Every claim derived from INTAKE in another spec file MUST cite:
```
Source: INTAKE 2026-05-22 [12:14]
```

This means: the speaker said this at 12:14 in the 2026-05-22 interview. Future agents can re-verify against the verbatim transcript when models improve.

## Agent contract for this file

- READ: any agent may read this file
- WRITE: only the Synthesizer (M-010) writes the initial intake. Subsequent follow-up interviews append a new section dated `## INTAKE — <client> — <date> (follow-up #N)`.
- NEVER: fabricate, paraphrase, or "clean up" content
- EDIT: only typo fixes to verbatim transcript and only when the speaker confirms

---

**Last updated:** 2026-05-15
**Last verified:** n/a — file is skeleton

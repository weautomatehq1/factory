# STATUS

Three sections only. Next agent reads this first.

## Done

### 2026-05-15 — Initial scaffold of The Factory repo
- Created `~/dev/coordination/factory/` with all 17 spec files
- Wrote 10 retroactive ADRs (0001–0010) capturing decisions 2026-05-12 → 2026-05-15
- Wrote MADR `_template.md`
- Established directory structure: root spec files + `docs/decisions/` + `.omc/`
- Set up `AGENTS.md` (cross-vendor convention) + `CLAUDE.md` (`@AGENTS.md` import bridge) per ADR-0001
- Skeleton files for `INTAKE.md`, `KPI.md`, `RUNBOOK.md` (explicitly empty per "correct not clever" principle)
- All spec files have YAML frontmatter with `status:` field

## In flight

- (none — initial scaffold complete, awaiting Sebastian's review)

## Up next

Once Sebastian approves the scaffold:

1. **Git init + push** — `cd ~/dev/coordination/factory && git init && git add . && git commit -m "feat: initial scaffold of The Factory master coordination repo" && create GitHub repo `weautomatehq1/factory` + push`
2. **Branch protection** — apply same ruleset as IFleet (no force-push, conversation resolution required)
3. **Start SPRINT-2026-05-A D1 work:**
   - Sebastian: scaffold `spec-template` repo (M-001)
   - Esme: start IFleet Gap 1 decomposer (M-002)
4. **Send Esme the master brief** — Discord #ifleet, with `[Sebas]` prefix, includes link to this repo + her D1 tasks
5. **30-min EOD sync** to lock the 3 open decisions (already provisionally locked via ADRs 0005/0006, voice platform pending R1)

## Open questions (need Sebastian decision)

- Should this repo be public (like IFleet for branch-protection-on-free) or private?
- Should Esme be a co-owner of `weautomatehq1/factory` from day 1?
- Do we add CI (markdownlint, frontmatter validation) before pushing, or after?

---

**Last updated:** 2026-05-15 by this Claude session (initial scaffold)

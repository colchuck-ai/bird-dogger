# Session 010 — Judged outcome O004 Triage Speed

## Goal

Run the judge pass on
`docs/product/outcomes/J001-O004-triage-speed/README.md` (drafted in
session 009) against the Outcome / Risk / Requirement rubrics in the
`intent` skill, before going to review.

## Context loaded

- `docs/product/outcomes/J001-O004-triage-speed/README.md` — the draft
  under judgment.
- `docs/product/README.md` — product root for vertical trace.
- `docs/product/outcomes/J001-O001-slip-detection-lag/README.md` —
  sibling; slip indicators are an input to triage; RSK002 boundary.
- `docs/product/outcomes/J001-O002-coverage/README.md` — sibling;
  triage operates on items already in the active set; RSK004 vs
  O002-RSK003 boundary.
- `docs/product/outcomes/J001-O003-status-fidelity/README.md` —
  sibling; status confidence is an input to triage.
- `resources/previous-sessions/004-outcome-o003-judged.md`,
  `006-outcome-o001-judged.md`, `008-outcome-o002-judged.md` — prior
  judge precedent (CR-or-not, surface-naming patterns, single-
  mitigation fragility framing).
- `resources/previous-sessions/009-outcome-o004-drafted.md` — the
  draft session's standing judge prompts.
- `intent` skill: `SKILL.md` (Outcome / Risk / Requirement rubrics),
  `references/context-tracing.md`, `references/workflows.md`,
  `assets/templates/outcome.md`.

Skipped: Records (CRs, PDRs, ADRs) per workflows guidance — judge on
a create cycle.

## Workspace path mismatch (still standing from sessions 002–009)

Cursor opens `/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`;
real repo is `/Users/max.dunn/dev/personal/colchuck-ai/
bird-dogging-skill` (resolved via symlink). Worked from the real
path. Worth fixing the workspace mapping.

## Verdict

Three material edits — all on the risk layer, all about scrubbing
product-shape vocabulary that belongs at the requirement layer.
Risk-Requirement Map unchanged. All seven standing prompts from S009
resolved without altering the draft's structural decisions.

## Standing judge prompts from S009 — resolutions

1. **RSK005 / R005 fold into RSK002 / R002 — keep distinct.** RSK002
   is item-level *attributes* (slip, freshness, due-date, blocker
   age) hidden behind per-item inspection. RSK005 is item-to-item
   *relationships* hidden. Different objects; different friction
   modes too (RSK002 = slowness-from-missing-info; RSK005 =
   mis-triage of un-actionable item). Same resolution pattern as
   O002-RSK003 vs O003-RSK003.
2. **R003 solution-talk pressure — pass.** "Synthesize urgency
   signals into a recommendation" is the *what*; the mechanism
   (heuristic, scoring, LLM, rule set) is the engineering layer.
   Same shape as O001-R001 and O002-R003.
3. **R004 "alongside what has changed" prescriptiveness — pass.**
   Describes the bird-dogger's information need (prior decision +
   delta), not how the delta is computed. Acceptable at the *what*
   level.
4. **R003 doubling on RSK001 + RSK003 — keep both links.** R001
   ranks within the flagged set; R003 *shrinks* the flagged set by
   committing to a threshold. Different mitigation paths. Different
   from S008's R001 demotion — that R001 only *showed* the watched
   set without solving the friction; here R003 actively reduces
   work the bird-dogger does.
5. **RSK004 vs O002-RSK003 boundary — keep distinct.** O002-RSK003
   (List Drop-Off) = the *item* leaves the active view. O004-RSK004
   (Triage Memory Loss) = the *triage decision* on the item is
   forgotten while the item is still on the list. Different
   objects.
6. **RSK002 (O004) vs O001-RSK002 boundary — keep distinct.**
   O001-RSK002 = single slip signal mixed among healthy items
   (haystack). O004-RSK002 = full urgency bundle hidden behind
   per-item inspection (depth). Different friction modes. The
   RSK002 reframe in this session sharpens the distinction.
7. **Single-mitigation fragilities (RSK002, RSK004, RSK005) — flag,
   not defect.** Parallel to O001-RSK002 / O002-RSK005 / O003-R005
   patterns. Documented standing cost.

## New issue the judge surfaced

**Surface-naming vocabulary leaked into the risk layer on three
risks.** RSK002 and RSK005 named "detail view / list level"; RSK003
named "the product" as the actor. None of the other risks across
O001 / O002 / O003 do this; the precedent ("data source," "tracked
data," "automated interpretation produces a status") is to describe
*conditions* without naming surfaces or the product. The fix was
small in each case — reframe the condition without surface
vocabulary; requirements R002 and R005 keep "list level" because the
requirement layer is the right place for product-surface
commitments.

## What got edited

`docs/product/outcomes/J001-O004-triage-speed/README.md`:

- **RSK002 reframed.** "...live inside the item's detail view rather
  than at the list level, requiring the bird-dogger to drill into
  items before recognizing what is urgent..." → "...are not visible
  alongside the items being triaged, so the bird-dogger must inspect
  each item individually before recognizing what is urgent..."
- **RSK003 reframed.** "The product surfaces urgency signals without
  committing to when those signals collectively cross into 'needs
  intervention now,'..." → "Urgency signals are available but no
  committed threshold defines when they collectively mean 'needs
  intervention now,'..."
- **RSK005 reframed.** "...are not surfaced at the list level, so
  the bird-dogger may mis-triage..." → "...are not visible alongside
  the items being triaged, so the bird-dogger may mis-triage..."
  Mirrors the RSK002 phrasing for consistency.

Risk-Requirement Map untouched.

No Change Record. Create + judge + review form one cycle; revisions
during judge are not a CR (S006 / S008 precedent, `workflows.md`).

## Coverage check post-edit

- Every risk has ≥1 requirement: ✓
- Every requirement has ≥1 risk: ✓
- All five risk impact clauses still close on the outcome statement
  (`...increasing the time to identify which items most need
  intervention right now.`). Preserved through all three edits.
- **R003 carries 2 risks** (RSK001 + RSK003). Defensible — distinct
  mitigation paths (set-shrinking vs threshold). Lower load than
  O001-R001 / O002-R001 pre-judge (3 risks each).
- **RSK002, RSK004, RSK005 each single-mitigation** (R002, R004,
  R005 respectively). Standing fragility, parallel to O001-RSK002 /
  O002-RSK005 / O003-R005 patterns. Not a defect.

## Vertical & horizontal trace

- Outcome statement lifted verbatim from product root — no drift.
- Outcome serves J001 (chase-loop step 3, between Cover and
  Escalate).
- Risks tie to this outcome's failure modes; requirements tie to ≥1
  risk on this outcome.
- Sibling boundaries (O001 / O002 / O003) hold — see prompts 5, 6,
  and the upstream-signal framing in S009.
- Cousin requirements: R002 consumes slip indicators (O001-R001),
  freshness (parallel to O003-R002 / O001-R002), due-date proximity,
  blocker age — adds to the pluggable-inputs engineering pressure
  S009 flagged. Nothing under O001 / O002 / O003 is contradicted.

## Deferred (still standing)

- **RSK002 / RSK004 / RSK005 single-mitigation fragilities.**
  Documented; review may probe.
- **R003 doubling on RSK001 + RSK003.** Argued distinct paths;
  review may try to demote R003 off RSK001.
- **R007 cross-outcome duplication** (Source Registration in O001
  and O002, identical wording). O004 doesn't reuse. Still standing
  from S008.
- **R005 fragility under O003** (lone preventive requirement,
  single-risk coverage) — still standing from S004. Distinct R005
  there from this session's R005; naming clash, unrelated content.
- **O001-RSK002 single-mitigation fragility** (R001 only) — still
  standing from S006.
- **Engineering concerns** sharpened by O004:
  - **Pluggable inputs** — R002 (List-Level Urgency Signals)
    consumes slip indicators, freshness, due-date proximity,
    blocker age, building on O001-R007 and O002-R002 / R005 / R007.
  - **Local-LLM decisioning** — R001 (Urgency Ranking) and R003
    (Intervention Recommendation) are the strongest product-layer
    signal yet that a decisioning component is implied at
    architecture.
  - **Decision cache keyed by issue-data hash** — R004 (Triage
    State Carry-Forward) reads directly onto this; cached triage
    state keyed by issue data hash gives "alongside what has
    changed" for free.
  - **Pluggable outputs** — not advanced by O004.
- **Vim swap files** `.README.md.swp` present in *both* the O003
  folder (flagged sessions 004 / 005 / 006 / 007 / 008 / 009) and
  the O004 folder (new this session — file dated 09:58 against
  README dated 09:14, so the README on disk is older than the swap
  file's last touch). Worked from on-disk per prior precedent.
  Author should reconcile both before committing.
- **Workspace path mismatch** in Cursor.

## Recommended next step

1. Branch + PR these edits so an independent reviewer can run
   session 011 (review) against the same Outcome / Risk /
   Requirement rubrics. Reviewer-fruitful probes:
   - The RSK002 / RSK005 reframes — did the surface-naming scrub
     drop any useful specificity, or are the new versions cleaner?
   - R003 doubling on RSK001 + RSK003 — does R001 alone cover
     RSK001?
   - R003 solution-talk (still a standing flag from S009).
   - R004 "alongside what has changed" (still a standing flag).
   - R005 vs R002 fold question, one more time.
2. After review on O004, draft **O005 Escalation Quality** — the
   last outcome. S002's flag about O005 packing both targeting and
   timing still stands; may need to split or argue the joint
   framing during drafting.

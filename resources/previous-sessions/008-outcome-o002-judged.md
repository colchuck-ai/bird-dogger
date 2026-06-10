# Session 008 — Judged outcome O002 Coverage

## Goal

Run the judge pass on
`docs/product/outcomes/J001-O002-coverage/README.md` (drafted in
session 007) against the Outcome / Risk / Requirement rubrics in the
`intent` skill, before going to review.

## Context loaded

- `docs/product/outcomes/J001-O002-coverage/README.md` — the draft
  under judgment.
- `docs/product/README.md` — product root for vertical trace.
- `docs/product/outcomes/J001-O001-slip-detection-lag/README.md` —
  sibling outcome (shares source-registration territory).
- `docs/product/outcomes/J001-O003-status-fidelity/README.md` —
  sibling outcome (freshness terminology overlap).
- `intent` skill: `SKILL.md` (Outcome / Risk / Requirement rubrics),
  `references/workflows.md`, `references/context-tracing.md`,
  `assets/templates/outcome.md`, `assets/templates/cr-simple.md`.

Skipped: prior session digests (001–007) — read only 006 and 007 to
pick up the recent thread and the standing judge prompts from S007.
Should have read all of them; missed at least one precedent (see
"Process slip" below).

## Workspace path mismatch (still standing from sessions 002–007)

Cursor opens `/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`;
real repo is `/Users/max.dunn/dev/personal/colchuck-ai/
bird-dogging-skill` (resolved via symlink). Worked from the real
path. Worth fixing the workspace mapping.

## Verdict

Material defects on the risk-requirement map and one coverage gap.
Three judge-time edits applied to O002, one editorial edit to the
product README. Net: O002 is tighter than it was post-draft.

## Standing judge prompts from S007 — resolutions

1. **RSK005 distinctness pressure — keep.** RSK005 (Active Set
   Drift) is about freshness of *active-set composition*; O001-R002
   is about freshness of *per-item slip status*. Different objects.
   No fold attempted; the rubric doesn't object. R006 stays as the
   sole mitigation; single-mitigation fragility is the documented
   cost.
2. **RSK001 vs RSK002 fold attempt — keep distinct.** RSK001 is
   "item exists nowhere as a record"; RSK002 is "item exists in a
   record we don't watch." Different triggers, different
   mitigations. No fold.
3. **R003 (Coverage Gap Discovery) solution-talk pressure — flipped
   the other way.** Original text ("candidate sources and candidate
   items that may be in scope") wasn't solution-talky; it was
   *under*-specified. Not verifiable without naming what "candidate"
   means. Rewrote to name the candidacy signals (reachability from
   watched data; presence in watched sources but excluded from the
   active set) — still at the *what* level, but now testable.
4. **R005 (Item Migration Linking) solution-talk pressure —
   acceptable.** "Follow an item when it moves between sources" is
   broad but reads as *what* the product promises, not *how* it
   tracks identity. Engineering layer decides identity mechanics.
   Kept as-is.
5. **R001 load concentration (3 risks) — collapsed.** This was the
   biggest map issue. R001 (Coverage Set Visibility) only *shows*
   the bird-dogger the watched set; it doesn't close RSK003 or
   RSK004 on its own (R004 and R005 do). Demoted R001 to RSK002
   only, where seeing the watched-source list is a genuine
   precondition for recognizing a gap.
6. **R006 vs O001-R002 distinctness — confirmed distinct.** Same
   resolution as S007 predicted: composition-of-active-set vs
   per-item-slip-status. Different objects, no conflict.

## New issues the judge surfaced (not anticipated by S007)

1. **Source-registration gap on RSK002.** R003 surfaces candidate
   sources but nothing in O002 actually lets the bird-dogger
   register one. The capability exists at J001-O001-R007. Added
   J001-O002-R007 with identical wording so sibling outcomes agree.
   Cousin-link is by-convention (identical text) rather than an
   explicit cross-reference; future move could be to lift this to a
   shared requirement once the architecture stage exists.
2. **RSK001 → R003 mapping is broken.** R003 finds items that exist
   in some source. An implicit verbal commitment has no source. R3
   can't see it. Dropped from RSK001's mapping. R002 (Manual Item
   Capture) is the sole and sufficient mitigation.
3. **"Checkpoint" was undefined.** Heavily used in O002 (outcome
   statement, RSK001/003/005, R001/004/006); never defined. Added
   one-paragraph definition under J001 in the product README,
   because the chase loop concept attaches at the job level and
   future outcomes may use the term.

## RSK002 ↔ O001-RSK005 boundary — confirmed resolved (from S007)

S007 set this; the judge confirmed it:

- **O001-RSK005 (Source Blind Spot)** — source is needed but the
  product can't read it (outage, no integration).
- **O002-RSK002 (Unwatched Source)** — source has not been
  registered for coverage.

Distinct failure modes. Same resolution pattern as O001-RSK005 vs
O003-RSK003.

## RSK003 ↔ O003-RSK003 boundary — confirmed resolved (from S007)

S007 set this; the judge confirmed it. Shared trigger (premature
"closed" flip), distinct downstream failures (fidelity vs coverage).
Mitigations differ.

## What got edited

`docs/product/outcomes/J001-O002-coverage/README.md`:

- **R003 tightened.** "Candidate sources and candidate items that
  may be in scope but are not currently being watched" →
  "Candidate sources — sources referenced by watched items or
  otherwise reachable from watched data but not themselves
  watched — and candidate items — items present in watched sources
  but excluded from the active set." Verifiable now.
- **R007 added.** "Source Registration: The product must let the
  bird-dogger register a new data source for tracking items."
  Wording identical to J001-O001-R007 so siblings agree.
- **Risk-Requirement Map rewritten.** Changes:

  | Risk | Before | After |
  |---|---|---|
  | RSK001 Implicit Commitment | R002, R003 | R002 |
  | RSK002 Unwatched Source | R001, R003 | R001, R003, R007 |
  | RSK003 List Drop-Off | R001, R004 | R004 |
  | RSK004 Item Migration Loss | R001, R005 | R005 |
  | RSK005 Active Set Drift | R006 | R006 (unchanged) |

`docs/product/README.md`:

- **"Checkpoint" defined under J001.** "A *checkpoint* is a
  recurring moment when the bird-dogger reviews the active set of
  open items to decide what needs intervention. The chase loop runs
  from one checkpoint to the next."

No Change Record. Create + judge + review form one cycle (S006
precedent); revisions during judge are not a CR. See "Process slip"
below.

## Coverage check post-edit

- Every risk has ≥1 requirement: ✓
- Every requirement has ≥1 risk: ✓
- All five risk impact clauses still close on the outcome statement
  (`...increasing the likelihood that the item is forgotten between
  checkpoints.`).
- **R001 load reduced from 3 risks to 1.** Standing fragility flag
  from S007 is closed; R001 is now a clean precondition for RSK002.
- **RSK005 → R006 only** (single-mitigation). Standing fragility,
  parallel to O001-RSK002 / O003-R005's single-coverage positions.
  Not a defect.
- **R001 single-risk coverage** (new this session, by-design). If
  R001 is ever weakened, RSK002 loses a precondition but still has
  R003 + R007 as direct mitigations. Less fragile than the prior
  3-risk load.

## Process slip

Mid-session, I drafted
`docs/product/outcomes/J001-O002-coverage/crs/
J001-O002-CR001-judge-feedback-incorporation.md` claiming "material
update to an existing outcome." That misread the workflow: O002 was
just created in S007; we're still in the create cycle. S006's
precedent (and `workflows.md`: "for create, there is no prior
element state... create does not produce a Change Record") rules
this out. Deleted the file and the empty `crs/` directory before
writing this digest. Lesson: when in doubt about CR-or-not, the
question is "was the element settled before this change?" — not
"was the change material?" Material judge-time revisions inside the
create cycle still aren't CR-eligible.

## Deferred (still standing)

- **RSK002 → R001 + R003 + R007 — three-way mitigation, no single
  fragility.** New shape this session; replaces the old "RSK002
  single-mitigation under O001-R001" flag (which referred to
  O001's RSK002, not this one — distinct risks).
- **R007 cross-outcome duplication.** Identical to J001-O001-R007.
  Future edits to either must mirror, or promote to a shared
  element at the architecture stage. Flag: drift risk.
- **R005 (Item Migration Linking) solution-talk pressure.** Still
  standing from S007 — not addressed this session. Review may
  probe again.
- **R005 fragility under O003** (lone preventive requirement,
  single-risk coverage) — still standing from S004.
- **O001-RSK002 single-mitigation fragility** (R001 only) — still
  standing from S006.
- **Engineering concerns** from sessions 001 / 002: pluggable
  tracker inputs, pluggable cache, pluggable outputs, local-LLM
  decisioning, decision cache keyed by issue-data hash. R007 now
  exists in *two* outcomes (O001 and O002), strengthening the
  pluggable-inputs pressure at the engineering layer. R002 (Manual
  Item Capture) and R005 (Item Migration Linking) under O002 also
  point at pluggable inputs. All attach as components once the
  architecture document exists.
- **Vim swap file** `.README.md.swp` in the O003 folder — flagged
  in sessions 004 / 005 / 006 / 007; not touched here. Author
  should reconcile.
- **Workspace path mismatch** in Cursor.

## Recommended next step

1. Branch + PR these edits so an independent reviewer can run
   session 009 (review) against the same Outcome / Risk /
   Requirement rubrics. Reviewer should re-probe the R005
   solution-talk question and the R007 cross-outcome duplication
   pattern.
2. After review on O002, draft **O004 Triage Speed** next — chase-
   loop step three, more independent than O005 per S007's framing.

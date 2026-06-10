# Session 013 — Drafted requirement documents for O001 Slip Detection Lag

## Goal

Use the `intent` skill to draft the per-requirement documents for
the most important outcome under J001. First session moving below
the outcome layer — all five outcomes are drafted, judged, and
(per S012's recommended next step) due for review, but the user
chose to push down into requirement documents for one outcome
rather than wait.

## Context loaded

- `docs/product/README.md` — product root; J001 + five outcomes.
- `docs/product/outcomes/J001-O001-slip-detection-lag/README.md` —
  outcome under drafting; requirement statements lifted verbatim
  for trace.
- `docs/product/outcomes/J001-O002-coverage/README.md` — sibling;
  R007 (Source Registration) appears here too with identical
  wording; relevant to R007 duplication call.
- `docs/product/outcomes/J001-O003-status-fidelity/README.md` —
  sibling; R002 (Status Freshness) is the boundary check against
  this outcome's R002 (Assessment Freshness) — different objects
  (status assessment time vs source data update time).
- `docs/product/outcomes/J001-O004-triage-speed/README.md` —
  sibling cousin; consumes R001's surfaced slipping set as urgency
  input.
- `docs/product/outcomes/J001-O005-escalation-quality/README.md` —
  sibling cousin; consumes slip status as escalation timing input.
- `resources/previous-sessions/005-outcome-o001-drafted.md`,
  `006-outcome-o001-judged.md` — original O001 framing decisions;
  standing flags (RSK002 single-mitigation fragility, R007
  cross-outcome duplication).
- `resources/previous-sessions/011-outcome-o005-drafted.md`,
  `012-outcome-o005-judged.md` — latest session digests for style
  and standing-flag carry-forward.
- `resources/bird-dogging/bird-dogging-overview.md` — bird-dogger
  responsibilities; slip detection as the keystone of "drive
  without doing the work yourself."
- `intent` skill: `SKILL.md` (Requirement rubric),
  `references/context-tracing.md` (Requirement trace —
  self/siblings/cousins), `references/workflows.md` (drafting
  session in a create cycle, no CR), `assets/templates/
  requirement-simple.md`, `assets/templates/requirement-nested.md`.

Skipped: Records (CRs, PDRs, ADRs) per workflows guidance — no
record history needed for a create; nothing in scope this session
warranted opening a PDR.

## Workspace path mismatch (still standing from sessions 002–012)

Cursor opens `/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`;
real repo is `/Users/max.dunn/dev/personal/colchuck-ai/
bird-dogging-skill` (resolved via symlink). Read tool failed on
the workspace path early in the session and was retried on the
real path; Write was issued against the real path from the start
to match S011/S012 precedent. Still worth fixing the workspace
mapping.

## Outcome selection

User asked for "the most important outcome." Pre-survey of all
five against three lenses (chase-loop position, product
differentiation, downstream feed):

- **O001 Slip Detection Lag** — chase-loop step 1 (Detect); what
  makes the product different from a plain list; feeds every
  downstream outcome; largest requirement set (7).
- **O002 Coverage** — precondition to O001; if items aren't on
  the list they can't be assessed. Runner-up.
- **O003 / O004 / O005** — all consume slip signal; valuable but
  derivative.

Recommended **O001**. User confirmed via AskQuestion. No
alternative survey beyond the pre-presented five options.

## Template choice

Simple template for all 7. None of the requirements carry a PDR,
CR, or other attached material at this point in the project.
Nested form (`.../R<NNN>-<name>/README.md`) would create empty
`pdrs/` and `crs/` subdirectories that read as bureaucratic
overhead. Convert per-requirement when the engineering layer or
a PDR attaches.

## Requirement framing decisions

Statement lines lifted verbatim from the outcome README's
requirements list — no rewording. Same pattern as outcome
statements being lifted verbatim from the product root
(S003/S005/S007/S009/S011). Drafting work happened in Detail,
Edge Cases, Examples, Acceptance Criteria, Dependencies.

Cross-cutting moves applied to all 7:

- **"Cannot assess" as a first-class state.** Used in R001, R004,
  R006. Default behaviour for absence-of-signal is to surface
  the gap, not silently equate it with on-track. Implicitly
  commits the product to a coverage-over-precision posture.
  Reviewer-fruitful — could be lifted to a product-root PDR
  rather than repeated across requirement docs.
- **Acceptance criteria stay solution-free.** No UI nouns, no
  channel choices, no thresholds, no model mentions. Verifiable
  but behavioral. Same defense applied to R004 in S010 for
  O004-R003.
- **Edge cases bias toward fail-safe defaults.** Every doc
  carries at least one edge case where the "we don't know"
  reading is forced to surface explicitly rather than collapse
  to on-track.

Per-requirement notes:

- **R001 (Proactive Slip Surfacing) — the hub.** Dependency
  list names R002, R004, R005, R006 as inputs. Risk: dependency
  vs adjacency line is fuzzy — R002 (freshness) is more
  *displayed alongside* than *consumed by*. Judge may try to
  narrow R001's dependency list. Argued: freshness travels with
  the surfaced item per the AC, so it's a consumption
  relationship at the surfaced-set layer.
- **R002 (Assessment Freshness) — boundary against O003-R002.**
  Detail explicitly disambiguates: this freshness is
  *product-assessment* time; O003-R002 is *underlying-source-
  data* time. Different objects, same shape. Judge precedent in
  S012 on O005-RSK002 vs O003-RSK002 (different objects, keep
  distinct).
- **R003 (Expected Trajectory) — "no trajectory" as a recorded
  state.** Explicitly distinct from "not yet recorded." Tightens
  R004's "cannot assess from trajectory" output. Judge may try
  to collapse the two states.
- **R004 (Trajectory Comparison) — finite state set in AC.** AC2
  forces the comparison output into a defined finite set rather
  than free text. Bounds the engineering layer's degrees of
  freedom; reviewer may want this softened to "a defined set"
  without "finite."
- **R005 (Silence As Signal) — unspecified silence window.**
  "Sustained absence" left unparameterized at the requirement
  layer; the window's inputs (item type, prior cadence, recorded
  trajectory) are flagged as engineering-layer concern. Judge
  may push for verifiability — is "sustained" testable without a
  number?
- **R006 (Source Availability) — per-item granularity.** Edge
  cases force per-item rather than source-wide unavailability,
  so partial failures don't silently degrade the whole source's
  items into on-track. Adds engineering-layer demand:
  unavailability is item-attached, not just source-attached.
- **R007 (Source Registration) — cross-outcome duplication
  acknowledged in-doc.** Detail section says outright "identical
  wording to J001-O002-R007; intentional; single capability
  serves both; standing cross-outcome flag for engineering
  layer." Carries the S008 standing flag into the per-
  requirement doc rather than letting it disappear at this
  layer. Judge may push to lift this into a product-root PDR
  instead of repeating in the doc.

## Boundary calls worth flagging for the judge

- **R001 dependency edges.** R002 listed as dependency may be
  more "displayed-alongside." Judge may narrow.
- **R002 vs O003-R002.** Different objects; precedent in S012.
  Judge will probe.
- **R003 "no trajectory" vs "not yet recorded".** Two distinct
  states; judge may try to collapse.
- **R004 finite state set in AC.** May be softened.
- **R005 unspecified silence window.** Verifiability pressure
  against the Requirement rubric ("verifiable"); engineering-
  layer-defers the parameter, but acceptance criteria avoid
  numbers. Reviewer-fruitful.
- **R006 per-item unavailability.** Tightens engineering
  demand; not a defect, but worth surfacing.
- **R007 in-doc duplication note.** Move to PDR vs keep in doc.

## What got written

Seven files under
`docs/product/outcomes/J001-O001-slip-detection-lag/
requirements/`:

- `J001-O001-R001-proactive-slip-surfacing.md`
- `J001-O001-R002-assessment-freshness.md`
- `J001-O001-R003-expected-trajectory.md`
- `J001-O001-R004-trajectory-comparison.md`
- `J001-O001-R005-silence-as-signal.md`
- `J001-O001-R006-source-availability.md`
- `J001-O001-R007-source-registration.md`

All simple template. Each: lifted statement, Detail, Edge Cases,
Examples (one scenario each), Acceptance Criteria (3 conditions
each), Dependencies (intra-outcome only — no cross-outcome
dependency edges asserted at this layer despite R007's
duplication note).

No Change Record — drafting session in a create cycle (S005,
S007, S009, S011 precedent and `workflows.md`).

## Coverage check

- Every requirement listed in the outcome README has a per-
  requirement document: ✓ (7 of 7).
- Every per-requirement document's statement matches the outcome
  README verbatim: ✓ (intentional, trace-preserving).
- Dependency graph forms a connected DAG (R001 ← {R002, R004,
  R005, R006}; R004 ← R003; R006 ← R007; R002 ← {standalone};
  R007 ← {standalone}): ✓. No cycles.
- Every dependency edge asserted in a per-requirement doc is
  internally consistent with the outcome README's risk-
  requirement map (no edge implies an unmapped risk).

## Deferred (so we don't lose it)

New this session:

- **R001 dependency vs adjacency line.** Judge will probe; argued
  consumption-at-the-surfaced-set-layer.
- **R004 finite-state-set AC.** May soften.
- **R005 silence-window verifiability without a number.** Most
  likely judge fruitful probe.
- **R007 in-doc duplication note vs product-root PDR.** Move-or-
  keep call.
- **"Cannot assess" repeated across R001/R004/R006.** Candidate
  for product-root PDR rather than repetition.

Carried forward:

- **O001-RSK002 single-mitigation fragility (R001 only).** Still
  standing from S006; now visible in R001's coverage role since
  it's the only mitigation for buried slip signal.
- **R007 cross-outcome duplication** (Source Registration in
  O001 and O002, identical wording). Still standing from S008;
  now visible in the per-requirement doc.
- **R005 fragility under O003** (lone preventive requirement,
  single-risk coverage). Still standing from S004; naming
  clash with O005's R005 and now also this outcome's R005 —
  three R005s in the tree; coincidence, distinct content.
- **No-owner-at-all seam between O002 and O005.** Still
  standing from S012.
- **Engineering concerns** sharpened by R001 as a signal-fan-in
  hub:
  - **Pluggable inputs** — R007 widens the input set; R001 fans
    in from R002/R004/R005/R006. Strongest product-layer
    statement of pluggable-input pressure to date.
  - **Decisioning component** — R001's "judges to be slipping"
    is the surface where the engineering layer must commit to a
    method (rule, heuristic, scoring, local-LLM). Joins
    O004-R001/R003 and O005-R004.
  - **Per-item state with freshness** — R002 demands per-item
    assessment timestamps separate from underlying data
    timestamps; reads naturally onto a per-item cache pattern
    (parallel to O004-R004, O005-R005).
  - **Per-item source-availability attribution** — R006's per-
    item granularity demand. New shape this session.
- **Workspace path mismatch** in Cursor.

Cleared:

- **Vim swap files** — still cleared per S011.

## Recommended next step

1. Branch + PR these seven new requirement docs so an independent
   reviewer can run session 014 (judge pass) on them against the
   Requirement rubric.
2. Judge pass — likely-fruitful prompts (in priority order):
   - R005 silence-window verifiability without a number.
   - R001 dependency edges (R002 in particular: dependency vs
     adjacency).
   - R003 "no trajectory" vs "not yet recorded" — collapse or
     keep distinct?
   - R007 in-doc duplication note — lift to product-root PDR?
   - "Cannot assess" repetition across R001/R004/R006 — lift to
     PDR?
   - R002 boundary against O003-R002.
   - R004 finite-state-set AC.
3. After judge + review on O001's seven requirement docs:
   - Either repeat the drafting cycle for O002 / O003 / O004 /
     O005 requirements (parallelizable across reviewers), or
   - Promote the cross-cutting flags (cannot-assess, R007
     duplication, decisioning, pluggable inputs) to product-root
     PDRs before drafting the engineering layer.
   - The PDR-first path likely saves rework — three of the
     cross-cutting flags would otherwise repeat across all 5
     outcomes' requirements.

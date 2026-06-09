# Session 007 — Drafted outcome O002 Coverage

## Goal

Use the `intent` skill to draft the next outcome document under J001.
Picked O002 Coverage on every prior session's standing recommendation
(sessions 005 and 006 closed with "draft O002 next" to resolve the
O001 ↔ O002 boundary).

## Context loaded

- `docs/product/README.md` — current product root.
- `docs/product/outcomes/J001-O001-slip-detection-lag/README.md` —
  drafted in session 005, judged in session 006. Sibling that shares
  boundary territory with O002 (the deferred item from S5/S6).
- `docs/product/outcomes/J001-O003-status-fidelity/README.md` —
  drafted in session 003, judged in session 004. Sibling whose
  drop-off / status-flip triggers can shade into O002's coverage
  territory.
- `resources/previous-sessions/001` through `006` — locked decisions,
  deferred risks, the standing O001 ↔ O002 boundary question, and
  the standing engineering deferrals.
- `resources/bird-dogging/bird-dogging-overview.md` — the practice.
- `intent` skill: `SKILL.md` (Outcome rubric, risk/requirement
  patterns), `references/context-tracing.md` (Outcome vertical /
  horizontal trace), `references/workflows.md` (three-session model),
  `assets/templates/outcome.md`.

Skipped: Records (CRs, PDRs, ADRs) per workflows guidance — no
record history needed for a create.

## Workspace path mismatch (still standing from sessions 002–006)

Cursor opens `/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`;
real repo is `/Users/max.dunn/dev/personal/colchuck-ai/
bird-dogging-skill` (resolved via symlink). Wrote against the real
path. Worth fixing the workspace mapping.

## Outcome selection

Surveyed the three remaining outcomes. O002 won on three grounds:

1. **Standing recommendation across S5 and S6.** Both prior sessions
   closed with "draft O002 next" to resolve the O001 ↔ O002 boundary.
2. **Chase-loop order.** Detect (O001) → Cover (O002) → Triage (O004)
   → Escalate (O005). O002 is step two; without coverage, detection
   only sees what was remembered to track.
3. **Deferred risks already queued.** Session 001's "new tracker
   source not integrated" deferral partly landed under O001-RSK005
   (Source Blind Spot), but its coverage flavor — items existing in
   places we never started watching — sits more cleanly here.

Credible alternatives surfaced and ruled out: O004 Triage Speed
(testing whether triage collapses into O001 + O003) and O005
Escalation Quality (forcing the targeting-vs-timing split early).
User confirmed O002.

## Outcome framing decisions

Drew the outcome line at "items not in the bird-dogger's active
set at all." Distinct from O001 (items in the set, slip not
noticed) and O003 (items in the set, status wrong).

## Risk framing decisions

Five risks, each customer-centric (the bird-dogger's coverage gap,
not system internals) and solution-free:

- **RSK001 Implicit Commitment.** Commitment lives in chat,
  meeting, or doc; never recorded as a trackable item. Covers both
  "never formed as a discrete item" and "exists only in
  unstructured form." Distinct from RSK002 because nothing exists
  to be watched, regardless of which sources are registered.
- **RSK002 Unwatched Source.** Item lives in a data source that has
  not been registered for the bird-dogger's coverage. Distinct from
  O001-RSK005 (see boundary resolution below).
- **RSK003 List Drop-Off.** Item leaves the active view (closed,
  archived, filtered, aged out) while real work remains. Distinct
  from O003-RSK003 (Lagging Tracker Data) because that's a
  *fidelity* failure (status disagrees with reality); this is a
  *coverage* failure (item is no longer on the list). Triggers can
  overlap; downstream failures are distinct.
- **RSK004 Item Migration Loss.** Item moves between
  sources/trackers and the bird-dogger watches the old location.
  Distinct from RSK002 (unwatched source) because here the new
  source may itself be watched — the failure is that the
  bird-dogger's pointer at the *item* points at the wrong record.
- **RSK005 Active Set Drift.** Active set not refreshed against
  watched sources between checkpoints; newly created items don't
  appear. Could fold as "implicit product mechanics"; argued
  distinct because freshness of *active-set composition* is
  separable from freshness of *per-item slip status* (O001-R002).
  Judge will probe.

## RSK002 ↔ O001-RSK005 boundary — resolved

This was the standing deferred item from S5 and S6. After drafting
O002:

- **O001-RSK005 (Source Blind Spot)** — source is *needed* but the
  product cannot read it (outage, integration not built). Detection
  fails on items we know belong there.
- **O002-RSK002 (Unwatched Source)** — source has *not been
  registered* for coverage; the bird-dogger doesn't know to watch
  it, or hasn't added it. Coverage fails because items in that
  source aren't in scope at all.

Triggers can overlap. Failure modes are distinct (detection vs
coverage). Same resolution pattern as O001-RSK005 vs O003-RSK003.
No cross-reference added in either README; the IDs and prose are
distinct enough.

## RSK003 ↔ O003-RSK003 boundary — resolved as parallel risks

Tracker prematurely flips an item to "closed" is a shared trigger:

- **O003-RSK003 (Lagging Tracker Data)** — status is wrong;
  fidelity failure.
- **O002-RSK003 (List Drop-Off)** — item drops out of the active
  set; coverage failure.

Two distinct downstream failures from the shared trigger. Both
risks stand under their respective outcomes. Mitigations differ
(O003-R004 status override vs O002-R004 drop-off surfacing); the
affordances overlap in practice but the framings are distinct.

## Requirement framing decisions

Six requirements, each tied to ≥1 risk. All behavioral, not
implementation:

- **R001 Coverage Set Visibility.** Load-bearer (3 risks: RSK002,
  RSK003, RSK004). Symmetric with O001-R001 / O003-R001-R002
  carrying outcome-level load. Watch for judge pressure.
- **R002 Manual Item Capture.** Sole mitigation for "item exists
  nowhere" (RSK001 partial). Implies pluggable inputs at the
  engineering layer — second such product-level signal after
  O001-R007.
- **R003 Coverage Gap Discovery.** The closest requirement to
  solution-talk in this outcome ("surface candidate sources and
  candidate items that may be in scope"). Argued as *what* the
  product must do, not *how*; the engineering layer decides
  scanning approach. Judge will probe.
- **R004 Drop-Off Surfacing.** Distinct from O003-R004 (status
  override). R004 here is about the coverage decision — item left
  the active set, surface it and let the bird-dogger keep it on
  the list. The affordances overlap (both can re-open an item from
  the bird-dogger's view), but the framings differ.
- **R005 Item Migration Linking.** Borders on engineering ("follow
  an item across sources" implies stable item identity). Argued as
  product-level: the *promise* is that coverage tracks the live
  item; the *implementation* is engineering. Judge will probe.
- **R006 Active Set Freshness.** Symmetric to O001-R002
  (Assessment Freshness). Different objects: O001-R002 is per-item
  slip-status freshness; O002-R006 is composition-of-the-active-set
  freshness. Distinct.

Risk-Requirement Map:

- RSK001 → R002, R003
- RSK002 → R001, R003
- RSK003 → R001, R004
- RSK004 → R001, R005
- RSK005 → R006

## What got written

`docs/product/outcomes/J001-O002-coverage/README.md` — full outcome
document with the outcome statement (lifted verbatim from the
product root for narrative coherence, same move as S3 and S5),
five risks, six requirements, and the risk-requirement map.

No Change Record — this is a create, per workflows.md.

## Coverage check

- Every risk has ≥1 requirement: ✓
- Every requirement has ≥1 risk: ✓
- All five risk impact clauses close on the outcome statement
  (`...increasing the likelihood that the item is forgotten between
  checkpoints.`). Pattern preempted from session 004's late fix on
  O003.
- **R001 carries 3 risks.** Load concentration intentional, parallel
  to O001-R001's 3-risk load. Defensible; if R001 is decomposed at
  the architecture stage, the decomposition must preserve all three
  coverage paths (RSK002 / RSK003 / RSK004).
- **RSK005 → R006 only** (single-mitigation). Standing fragility,
  parallel to O001-RSK002 / O003-R005's single-coverage positions.
  Not a defect.

## Deferred (so we don't lose it)

- **RSK005 distinctness pressure.** May fold into "implicit product
  mechanics" under judge. If folded, R006 needs a new home or
  drops; if kept, RSK005's single-mitigation fragility is the
  defensible cost.
- **RSK001 vs RSK002 fold attempt.** Both describe items missing
  from coverage; differ by trigger (no record vs unwatched source).
  Distinct in principle; judge may try to fold.
- **R003 (Coverage Gap Discovery) solution-talk pressure.** Closest
  to "how" of any requirement here. Argued *what*; expect pushback.
- **R005 (Item Migration Linking) solution-talk pressure.** Implies
  stable item identity. Argued *what*; expect pushback.
- **R001 load concentration (3 risks).** Standing flag, same as
  O001-R001.
- **R006 vs O001-R002 distinctness.** Different objects (active-set
  composition vs per-item slip status). Judge will probe.
- **Engineering concerns** from sessions 001 / 002 still standing:
  pluggable tracker inputs, pluggable cache, pluggable outputs,
  local-LLM decisioning, decision cache keyed by issue-data hash.
  R002 (Manual Item Capture) and R005 (Item Migration Linking) here
  add to the pluggable-inputs pressure already implied by O001-R007.
  All attach as components once the architecture document exists.
- **R005 fragility under O003** (lone preventive requirement,
  single-risk coverage). Still standing from session 004.
- **RSK002 single-mitigation fragility under O001** (R001 only).
  Still standing from session 006.
- **Vim swap file** `.README.md.swp` in the O003 folder — flagged
  in sessions 004, 005, 006; not touched here. Author should
  reconcile.
- **Workspace path mismatch** in Cursor.

## Recommended next step

1. Branch + PR these changes (bundling any uncommitted prior-session
   edits if those are still pending) so an independent reviewer can
   run session 9 (judge pass) — wait, judge is session 8 here,
   review is session 9. Order: judge first, then review.
2. Run the judge pass on
   `docs/product/outcomes/J001-O002-coverage/README.md` against the
   Outcome / Risk / Requirement rubrics. Likely-fruitful prompts:
   - RSK002 vs O001-RSK005 boundary — validate the distinction.
   - RSK005 distinctness — keep, fold into R001's "show what's in
     the active set" implication, or drop entirely?
   - RSK001 vs RSK002 fold attempt.
   - R003 (Coverage Gap Discovery) solution-talk pressure.
   - R005 (Item Migration Linking) solution-talk pressure.
   - R001 load concentration (3 risks).
   - R006 vs O001-R002 distinctness.
3. After judge + review on O002, draft **O004 Triage Speed** next —
   it's chase-loop step three and the more independent of the two
   remaining outcomes (O005 packs targeting + timing per S2's flag,
   so leaving it last lets O004's risks settle first).

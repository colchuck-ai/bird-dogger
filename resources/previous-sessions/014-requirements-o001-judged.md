# Session 014 — Judged requirement documents for O001 Slip Detection Lag

## Goal

Run the judge pass on the seven per-requirement documents under
`docs/product/outcomes/J001-O001-slip-detection-lag/requirements/`
(drafted in session 013) against the Requirement rubric in the
`intent` skill, before going to review. First judge session below
the outcome layer.

## Context loaded

- All 7 R-docs under
  `docs/product/outcomes/J001-O001-slip-detection-lag/requirements/`
  — the drafts under judgment.
- `docs/product/outcomes/J001-O001-slip-detection-lag/README.md` —
  owning outcome; risk-requirement map for coverage check.
- `docs/product/README.md` — product root for vertical trace.
- `docs/product/outcomes/J001-O002-coverage/README.md` — sibling;
  R007 (Source Registration) duplication boundary; landing spot
  for the missing de-registration capability flagged here.
- `docs/product/outcomes/J001-O003-status-fidelity/README.md` —
  sibling; R002 boundary check (O001-R002 slip-status assessment
  vs O003-R002 item-status assessment + source-data update).
- `docs/product/outcomes/J001-O004-triage-speed/README.md` —
  sibling cousin; consumes R001 surfaced set + R002 freshness as
  urgency input.
- `docs/product/outcomes/J001-O005-escalation-quality/README.md`
  — sibling cousin; consumes R001 slip status as escalation
  timing input.
- `resources/previous-sessions/013-requirements-o001-drafted.md`
  — drafting session; the seven standing judge prompts and
  carried-forward flags.
- `resources/previous-sessions/012-outcome-o005-judged.md`,
  `010-outcome-o004-judged.md`, `008-outcome-o002-judged.md`,
  `006-outcome-o001-judged.md`, `004-outcome-o003-judged.md` —
  prior judge precedent (fold-vs-split resolutions, single-
  mitigation framing, signal-list defenses, boundary patterns).
- `intent` skill: `SKILL.md` (Requirement rubric),
  `references/context-tracing.md` (Requirement self/siblings/
  cousins), `references/workflows.md` (judge inside create cycle
  has no CR), `assets/templates/requirement-simple.md`.

Skipped: Records (CRs, PDRs, ADRs) per workflows guidance —
judge on a create cycle.

## Workspace path mismatch (still standing from sessions 002–013)

Cursor opens `/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`;
real repo is `/Users/max.dunn/dev/personal/colchuck-ai/
bird-dogging-skill` (resolved via symlink). Worked from the real
path; Read tool tried the workspace path first and failed, then
succeeded on the real path. Still worth fixing the workspace
mapping.

## Verdict

Soft edits on R001, R002, R006. R003, R004, R005, R007 hold as
drafted on precedent. Three new flags lifted to PDR-candidate
status (deferred). Not as clean a pass as S006 or S012; closer
to S010's shape — edits real but small, no defects.

## Standing judge prompts from S013 — resolutions

1. **R005 silence-window verifiability without a number — pass.**
   AC #1 names "beyond the product's silence window" without
   committing W. The test is parameterized on W; the requirement
   constrains shape (sustained absence ⇒ slip), not threshold.
   Same precedent as R001 (no channel/layout), R004 (no
   comparison method), O004-R003 (signal list without
   thresholds, upheld in S010). "Verifiable" in the rubric means
   the test runs once W is committed elsewhere, not that W lives
   in the requirement. ACs #2 and #3 are directly testable
   today. No edit.

2. **R001 dependency on R002 — soft edit. Dropped.** R001 Detail
   names R003/R004/R005 as inputs to the slipping judgement; R002
   (freshness) is not in that decision. R002 self-mandates its
   own display per its own AC #1. The S013 "freshness travels
   with the surfaced item" defense holds at the *layout* level
   but not at the *consumption* level. R006 stays — R001's
   "cannot assess" edge case consumes the source-unread state.
   Edited: R002 dropped from R001 Dependencies; R006's entry
   tightened to name the cannot-assess consumption explicitly.

3. **R003 "no trajectory" vs "not yet recorded" — keep distinct.**
   AC #3 makes the distinction load-bearing. The states drive
   different operator actions: "not yet recorded" is a coverage
   gap (R001 surfaces; bird-dogger sets); "no trajectory" is the
   explicit choice (R001 reads; bird-dogger acted). R004 collapses
   them only for *its own* output (comparison not run), which is
   correct scoping. No edit.

4. **R007 in-doc duplication note — defer to PDR session.** The
   note is defensible in-doc; the right long-term home is a
   product-root PDR ("Source Registration is one capability shared
   by O001 and O002; engineering realizes as a single component
   fulfilling both"). Not blocking. Flagged for a dedicated PDR
   session that batches with prompt 5.

5. **"Cannot assess" repetition across R001/R004/R006 — defer to
   PDR session.** Higher-fruit PDR than R007. The principle is a
   *posture* (coverage > precision: absence-of-signal never
   collapses to on-track), not a capability. Each requirement
   correctly asserts the posture for its own scope; the PDR names
   the shared commitment once. Deferred.

6. **R002 boundary against O003-R002 — soft edit on Detail
   phrasing.** Different objects confirms the keep-distinct
   verdict (precedent: S012 on O005-RSK002 vs O003-RSK002). But
   the current Detail framed the boundary as "product assessment
   vs source data update" — which only addressed half. O003-R002
   carries *both* a product-side assessment timestamp (item
   status) and a source-data update timestamp. Edited: Detail
   rewritten to name both legs distinct from this R002's
   slip-status assessment time.

7. **R004 finite-state-set AC — hold.** "Finite" is precise; the
   parenthetical `(e.g. on-track / slipping / cannot assess)`
   telegraphs intent; "bounded enumeration" would be a defensible
   swap but unnecessary. Reviewer-fruitful, not judge-fruitful.
   No edit.

## New issues the judge surfaced

**R005 ↔ R001 signal-reconciliation home — soft edit on R001.**
R005's third edge case names the case ("trajectory explicitly
allows a long quiet period") but says "R001 resolves the combined
reading." R001's pre-edit Detail and Edge Cases didn't address
signal-conflict — only insufficient signal ("cannot assess") and
state transitions. The split between "R005 names the case, R001
owns the resolution" read thin because R001 didn't acknowledge
ownership. Edited: R001 Detail gained a paragraph committing to
single combined per-item reading on signal disagreement, with the
reconciliation rule deferred to the engineering layer. R005's
edge case unchanged — it correctly defers to R001.

**R006 "permanently retired source" edge gap — soft edit on R006.**
The pre-edit edge case named "until the bird-dogger explicitly
removes it from coverage" but no requirement covered
de-registration. R007 owns registration only. Three options
considered:
- (a) Add an R008 de-registration requirement (likely O002).
- (b) Drop the retired-source edge case from R006.
- (c) Reframe to defer to a future source-management capability.

Picked (c) — least intrusive, honest about the gap. Edited:
R006 edge case reframed to surface the de-registration gap
explicitly and flag it for O002 alongside R007. Until that
capability exists, retired sources continue to surface as
unavailable rather than silently disappear. (a) is the more
honest long-term path and is now standing.

**R001 / R003 / R004 implicit history capability — flag, not
edit.** "Prior surfacing is recoverable in history" (R001),
"prior trajectory is recoverable" (R003), "prior comparisons
remain in history" (R004) all assert a history/audit capability
with no owning requirement. Less critical than R006's gap because
history is a *property* of multiple requirements rather than a
separate capability — naturally engineering-layer (audit/event
store). Carried forward as engineering concern.

**R003 trajectory adoption vs source refresh — internally coherent
but tight; no edit.** Edge case "adopt from source" + AC #2 "not
overwritten by source refreshes" resolves to "initial adoption
once; bird-dogger must re-baseline explicitly." Reads coherent.
Worth surfacing for review; reviewer may push for explicit
re-baseline contract.

## What got edited

Four edits, three files. Bullet form:

- `J001-O001-R001-proactive-slip-surfacing.md`
  - Detail: added a paragraph committing R001 to single combined
    per-item reading on signal disagreement; reconciliation rule
    deferred to engineering layer.
  - Dependencies: dropped R002 entry; tightened R006 entry to
    name the cannot-assess consumption explicitly.
- `J001-O001-R002-assessment-freshness.md`
  - Detail: rewrote O003-R002 boundary clause to name both legs
    (item-status assessment time + source-data update time)
    distinct from this R002's slip-status assessment time.
- `J001-O001-R006-source-availability.md`
  - Edge Cases: reframed retired-source bullet to defer
    de-registration to a future source-management capability;
    flagged for O002 alongside R007; explicit gap-not-silent-
    failure framing.

No Change Record — judge inside a create cycle (S006 / S008 /
S010 / S012 precedent, `workflows.md`).

## Coverage check

- Risk–requirement map intact: ✓ all five risks covered; all
  seven requirements mapped.
- Per-doc dependency edges form a connected DAG with R001 as hub:
  ✓ no cycles. R003 ↔ R004 reciprocal is acceptable (declared
  trajectory is the line; comparison is the read).
- Statement lines match outcome README verbatim: ✓
- All ACs solution-free (no UI nouns, no thresholds, no model
  mentions): ✓
- All statements follow rubric pattern `[Product/Solution] must
  [Capability/Constraint]`: ✓
- **RSK002 (Buried Slip Signal) — single-mitigation (R001
  only).** Standing fragility from S006.
- **R007 ≡ O002-R007 wording.** Standing from S008; now
  PDR-candidate (prompt 4).
- **R006 retired-source → de-registration gap.** New; soft-edit
  applied to surface it; (a)-path (R008 in O002) standing.

## Vertical & horizontal trace

- **Vertical:** every requirement traces to a mapped risk; every
  risk impact clause closes on the outcome statement.
- **Sibling (intra-outcome):** dependencies form a connected DAG
  with R001 as fan-in hub. R005's reconciliation edge case
  ownership now made explicit on R001's side.
- **Cousin (cross-outcome) — sharpened this session:**
  - **O001-R002 (slip-status assessment time) feeds O004-R002
    (list-level urgency facts).** Same per-item timestamp serves
    both outcomes' display.
  - **O001-R001 (surfaced slipping set) feeds O004-R001 (urgency
    ranking) and O004-R003 (intervention recommendation).** Slip
    status is upstream of triage.
  - **O001-R001 also feeds O005-R004 (escalation timing).** Same
    source; downstream of triage.
  - **O001-R007 (Source Registration) ≡ O002-R007.** Cross-outcome
    capability. PDR-candidate.
  - **O001-R002 vs O003-R002.** Same shape (per-item product-side
    assessment timestamp), different objects (slip status vs item
    status). Boundary now sharpened in R002 Detail.

## Deferred (so we don't lose it)

New this session:

- **R006 de-registration gap — (a)-path standing.** Soft edit
  applied to flag; the real fix is an R008 in O002 alongside R007.
  Reviewer-fruitful; engineering-layer-coupled.
- **R001 reconciliation rule — engineering-layer deferral.**
  R001 now owns the *requirement* (single combined reading); the
  *rule* is engineering. Will land as ADR or architecture detail.
- **History capability** (R001 / R003 / R004) unowned at
  requirement layer. Engineering-layer flag (audit/event store).
- **R003 trajectory re-baseline contract** — coherent but tight;
  reviewer-fruitful.

Promoted to PDR-candidates (deferred to dedicated PDR session):

- **"Cannot assess" posture** — coverage > precision; repeated
  across R001/R004/R006. Highest-fruit PDR.
- **R007 Source Registration** — capability shared by O001 and
  O002. PDR home for the cross-outcome decision.

Carried forward:

- **O001-RSK002 single-mitigation fragility** (R001 only).
  Standing from S006.
- **R007 cross-outcome duplication.** Standing from S008. Now
  PDR-candidate.
- **R005 fragility under O003** (lone preventive). Standing from
  S004. Three R005s in tree; coincidence, distinct content.
- **No-owner-at-all seam** between O002 and O005. Standing from
  S012.
- **Engineering concerns** (now sharpened by per-requirement
  layer):
  - **Pluggable inputs** — R007 widens input set; R001 fans in
    R004/R005/R006.
  - **Decisioning component** — R001's "judges to be slipping"
    plus the new combination rule; joins O004-R001/R003,
    O005-R004.
  - **Per-item state with freshness** — R002 demands per-item
    assessment timestamps separate from underlying data
    timestamps (and now distinct from O003-R002's two
    timestamps); reads onto a per-item cache pattern.
  - **Per-item source-availability attribution** — R006's
    per-item granularity demand; tightened this session by
    explicit de-registration gap flag.
- **Workspace path mismatch** in Cursor.

Cleared:

- (none) — vim swap files remain cleared per S011.

## Recommended next step

1. **Branch + PR** the three edited requirement docs (R001, R002,
   R006) plus the four unchanged ones, so an independent reviewer
   can run session 015 (review) against the Requirement rubric.
   Reviewer-fruitful probes:
   - R001 dependency rephrasing — was R002 dropped correctly, or
     should it be re-added as adjacency?
   - R001 reconciliation paragraph — does it commit too much, or
     should it commit more (e.g., signal-conflict edge case)?
   - R002 boundary phrasing — does the three-timestamp framing
     land?
   - R006 retired-source gap — accept (c) reframe, or push for
     (a) R008 in O002?
   - R005 silence window verifiability without a number.
   - R003 "no trajectory" vs "not yet recorded" distinction.
   - R004 finite-state-set AC (soft "bounded enumeration" swap).
   - R007 in-doc duplication note vs PDR.
   - "Cannot assess" repetition vs PDR.
2. **PDR session** — batch the "cannot assess" posture and R007
   cross-outcome decision into product-root PDRs before drafting
   more requirements or the engineering layer. Likely saves
   rework across O002–O005's eventual requirement drafts.
3. After review on O001, decision point:
   - Repeat drafting cycle for O002 / O003 / O004 / O005
     requirements (parallelizable across reviewers), **or**
   - Do the PDR session first (recommended — three of the
     cross-cutting flags would otherwise repeat across all 5
     outcomes' requirements).

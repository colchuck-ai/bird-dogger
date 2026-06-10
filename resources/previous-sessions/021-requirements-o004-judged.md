# Session 021 — Judged requirement documents for O004 Triage Speed

## Goal

Run the judge pass on the five per-requirement documents under
`docs/product/outcomes/J001-O004-triage-speed/requirements/`
(drafted in S020) against the Requirement rubric in the `intent`
skill, before going to review. Third judge session below the
outcome layer; S014 (O001) and S018 (O003) were the prior two.
First R-doc judge session after the three product PDRs landed —
PDR citations were S020's load-bearing format precedent.

## Context loaded

- All 5 R-docs under
  `docs/product/outcomes/J001-O004-triage-speed/requirements/`
  — the drafts under judgment.
- `docs/product/outcomes/J001-O004-triage-speed/README.md` —
  owning outcome; risk-requirement map for coverage check.
- `docs/product/README.md` — product root for vertical trace.
- All three PDRs:
  `docs/product/pdrs/PRD001-coverage-over-precision.md`,
  `PRD002-per-item-freshness-presentation.md`,
  `PRD003-shared-source-registration.md` — fully loaded for
  citation-correctness check (each R-doc references PRD001
  and/or PRD002 inline in Edge Cases per S020 precedent).
- Sibling outcome READMEs:
  `J001-O001-slip-detection-lag/README.md`,
  `J001-O002-coverage/README.md`,
  `J001-O003-status-fidelity/README.md`,
  `J001-O005-escalation-quality/README.md` — read for cousin
  context and downstream-consumer fit.
- Cousin requirement docs for every declared cross-outcome edge:
  - `J001-O001-R002` (assessment freshness) — declared dep of
    R002.
  - `J001-O002-R001` (coverage set visibility) — declared dep
    of R005.
  - `J001-O002-R004` (drop-off surfacing) — referenced in R004
    EC2.
  - `J001-O002-R006` (active-set freshness) — declared dep of
    R004; surfaced as missing dep on R005.
  - `J001-O003-R001` (status basis) — declared dep of R002.
  - `J001-O003-R002` (status freshness) — declared dep of R002
    and R004.
  - `J001-O003-R003` (status confidence) — declared dep of R003;
    surfaced as missing dep on R002.
  - `J001-O003-R004` (status override) — cross-referenced by
    R003 for override-shape pattern.
  - `J001-O003-R006` (conflict surfacing) — referenced in R001
    EC2 for conflicting-signals posture.
- `intent` skill: `SKILL.md` (Requirement rubric),
  `references/context-tracing.md` (Requirement self/siblings/
  cousins), `references/workflows.md` (judge inside create cycle
  has no CR), `assets/templates/requirement-simple.md`.
- `resources/previous-sessions/020-requirements-o004-drafted.md`
  — drafting session; the six standing judge prompts and the
  PDR-citation format precedent. Also S014 and S018 for prior
  judge precedent below the outcome layer (rubric application,
  soft-edit patterns, fold-vs-split resolutions, snapshot-vs-
  live-read framing).

Skipped: Records (CRs, ADRs) per workflows guidance — judge on
a create cycle.

## Workspace path mismatch — still standing, worked around

Seventh session running with the same workaround. Cursor opens
`/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`; real repo
is `/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. Read
tool calls against the workspace path failed on first attempt;
`pwd` confirmed the mismatch. Switched to absolute paths through
the real repo for all reads + writes — same workaround
S015/S017/S018/S019/S020 used. `git status` on `bird-dogger/`
on branch `product-docs` (tracking `origin/product-docs`) shows
the five modified R-doc files plus this digest as the only
changes from this session.

**Standing for next session:** unchanged — repoint the Cursor
workspace at
`/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. Seven
sessions deep on the workaround now.

## Verdict

Six soft edits across five files, all sharpening passes. No
defects; no held-back fragility beyond what S020 had already
flagged. Closer to S018's shape than S014's — judge surfaced
dependency-declaration gaps (R002 missing O001-R001 +
O003-R003; R005 missing O002-R006) and one symmetry gap (R001
EC didn't acknowledge dep-blocked items as a ranking input
even though R005 claimed to inform R001's ranking). Two
load-bearing scope calls (R003/R004 enum posture; R001 ↔ R005
asymmetry direction) resolved without AskQuestion — both had
clear precedent (R003 closed enum because override needs stable
target; add EC to R001 rather than drop the R005 claim).

One judgment-call recommendation was reconsidered mid-session
and skipped: an edit to RSK001's description in the O004
README. On second reading, RSK001's own wording
("requiring the bird-dogger to evaluate every flagged item to
choose what to act on first") already justifies the
RSK001 → R003 mapping — R003's recommendation reduces that
evaluation cost. The fix landed in R003's Detail (citing RSK001
alongside RSK003) rather than in the README description.

## Standing judge prompts from S020 — resolutions

1. **R003 override-shape vs. O003-R004 — hold parallel-pattern
   framing; do not promote to PDR.** Two override layers across
   two outcomes is not yet enough pressure for a fourth PDR
   (PRD001/PRD002 each had 3+ instances at the outcome layer
   before PDR pressure landed). R003's Detail already names
   the parallel explicitly ("survives re-synthesis on later
   checkpoints in the same shape O003-R004 commits to for
   status"). If O005-R004 (Escalation Timing Recommendation)
   introduces a third override layer on the same shape, the
   PDR pressure becomes real. Held for re-evaluation during
   O005 drafting. No edit.

2. **PDR-citation format — accept Edge Cases inline.** S020's
   choice (relative-path Markdown links into `pdrs/`, sitting
   next to the surfacing each citation justifies) reads
   correctly across all five R-docs. Treating PDRs as
   dependencies would conflate decisions with requirements
   (S020's argument). Format now precedent for O005. Reviewer
   may push for Dependencies-list citation; that's
   reviewer-fruitful, not judge-fruitful. No edit.

3. **R002 cross-outcome deps breadth — accept; in fact extend.**
   S020 flagged three cross-outcome edges (O001-R002, O003-R001,
   O003-R002) as "honest but heavy." Judge found the breadth is
   load-bearing — R002 is the list-level aggregator — and that
   the list was actually *short* relative to what Detail names.
   Extended to add O001-R001 (slip surfacing) and O003-R003
   (status confidence); see new issue N1. Five cross-outcome
   deps on R002 is the right number, not an over-count.

4. **R005 cycle-handling commitment — hold as edge case.** R005
   EC1 ("cycles surface as cycles per PRD001, not silently
   broken or hidden") plus AC #3 ("dependency cycles surface as
   cycles rather than being silently broken") together carry the
   commitment without a separate requirement. Same shape as
   O003-R006's "surface conflict, don't resolve" sitting as one
   requirement rather than splitting into "detect" + "surface."
   No edit.

5. **R001 "more than two buckets" granularity floor — hold.**
   Sharper than "differentiates" (which was the
   minimum-rubric phrasing) but still solution-free — no scale,
   no band count, no weighting. Same shape as O003-R003
   confidence-scale hold (S018 prompt 4). AC #1 verifiable today
   without prescribing the scale. No edit.

6. **R004 "what has changed" enumeration — hold.** Detail names
   "rank, signals, recommendation" as the recorded context;
   enumeration of "signals" is left to engineering. Verifiable
   through AC #2 (anchored on PRD002 shape). Same posture as
   R005 dependency-capture-mechanism deferral and O001-R005
   silence-window deferral. No edit on the enumeration — but
   see new issue N4 for a related snapshot-vs-live-read
   clarification edit on the same paragraph.

## New issues the judge surfaced

**N1. R002 Dependencies omits O001-R001 and O003-R003 despite
Detail naming them — soft edit applied.** R002 Detail lists the
signal set as "slip indicators (O001), status basis and
confidence (O003), per-item freshness, due-date proximity,
blocker age." Dependencies list named O001-R002 (assessment
freshness) but not O001-R001 (the slip surfacing itself);
named O003-R001 (basis) and O003-R002 (status freshness) but
not O003-R003 (confidence). Two mechanical gaps. Added both
to Dependencies with one-line consumption framing each. Same
precedent as S014 prompt 2 tightening R001's R006 entry to
name the consumption explicitly.

**N2. R001 ↔ R005 ranking-input asymmetry — soft edit applied
on R001.** R005's Dependencies declared "R001 — dependencies
inform the ranking; an item whose only path forward is blocked
should not rank above its blocker silently." R001 Detail/Edge
Cases did not acknowledge dep-blocked items as a ranking input
or commit to the no-silent-outranking shape — the claim lived
only on R005's side. Same shape as S014's R001 ↔ R005
signal-reconciliation gap (R005 named the case but R001 didn't
acknowledge ownership). Added an R001 Edge Case naming
dep-blocked items as a ranking input, with R005 named as the
declaration source. Direction matches S014's precedent: when
two requirements claim a shared shape, the hub side
acknowledges, the spoke side declares.

**N3. R003/R004 enum-posture inconsistency — soft edit applied
on R003 to explain, not flatten.** R003 AC #1 enumerates
exactly three states ("intervene now," "not now," "cannot
recommend"). R004 Detail says decision states are "illustrative;
R004 does not prescribe the closed set." Both are right for
their own scope but the contrast read like accident rather than
intent. Two options:
- (a) Tighten R003 to match R004's open vocabulary.
- (b) Loosen R004 to match R003's closed enum.
- (c) Explain the asymmetry in R003's Detail.
Picked (c). R003's three states must be closed because the
override (a product-side feature) needs a stable target and
the list-level renderer under R002 must show a known state;
R004's decision vocabulary belongs to the workflow (the
bird-dogger labels the decision, not the product), so an open
set is correct there. Edited: new paragraph in R003's Detail
making the asymmetry explicit and pointing at R004.

**N4. R004 record-time dependencies on R001/R002/R003
undeclared — soft edit applied on R004 to clarify, not declare.**
R004 Detail says it records "the context at the time of the
decision (rank, signals, recommendation)" — rank from R001,
signals from R002, recommendation from R003. Three implicit
intra-outcome dependencies at record-time, none declared. Two
options:
- (a) Add R001/R002/R003 to R004 Dependencies as record-time
  consumption.
- (b) Clarify in Detail that the recorded context is a
  snapshot R004 captures and stores; R004 does not re-read
  R001/R002/R003 live for prior decisions.
Picked (b). R004's "memory hub" framing depends on it being
acyclic with R003 (R003 reads R004's prior output, not the
reverse). Declaring three live deps would muddy that. Snapshot-
vs-live-read distinction is the honest characterization:
shape-of, not live-on. Edited: one sentence in Detail
explicitly making the distinction.

**N5. R003 → RSK001 mapping rationale was thin — soft edit
applied on R003 Detail, not on README.** Initial judgment
recommended tightening RSK001's description in the outcome
README to make the R001+R003 composition explicit. On second
reading, RSK001's wording ("requiring the bird-dogger to
evaluate every flagged item to choose what to act on first")
already pre-justifies R003 — the recommendation reduces that
evaluation cost. The actual gap was that R003's Detail cited
only RSK003. Fix landed in R003's Detail (first paragraph now
cites both RSK003 and RSK001's evaluation cost) rather than
in the README. Map unchanged.

**N6. R005 Dependencies missed O002-R006 — soft edit applied.**
R005 declared O002-R001 (coverage set visibility) as defining
"in-set" — but "in-set" between checkpoints depends on a
refreshed picture, which O002-R006 (active-set freshness)
maintains. Added O002-R006 with one-line framing. Same
consumption pattern as R004 → O002-R006 already declared.

## What got edited

Six edits, five files. Bullet form:

- `J001-O004-R001-urgency-ranking.md`
  - Edge Cases: added bullet for dep-blocked items not silently
    outranking their blockers (per R005's ranking-input claim).
    Closes N2 asymmetry.
- `J001-O004-R002-list-level-urgency-signals.md`
  - Dependencies: added O001-R001 (slip surfacing) and
    O003-R003 (status confidence). Closes N1 mechanical gap.
- `J001-O004-R003-intervention-recommendation.md`
  - Detail (¶1): tweaked to cite RSK001's evaluation cost
    alongside RSK003 — makes the R001+R003 composition behind
    the README map explicit. Closes N5.
  - Detail (new ¶ after the "engineering-layer" paragraph):
    explains why the three recommendation states are a closed
    set (override needs stable target; renderer needs known
    state) and contrasts with R004's intentionally-open
    decision vocabulary. Closes N3.
- `J001-O004-R004-triage-state-carry-forward.md`
  - Detail (extension of the "what has changed" paragraph):
    clarifies that the recorded rank/signals/recommendation
    context is a snapshot R004 captures at decision-time and
    stores; R004 does not re-read R001/R002/R003 live for
    prior decisions. Closes N4 without declaring three new
    intra-outcome deps.
- `J001-O004-R005-inter-item-dependency-surfacing.md`
  - Dependencies: added O002-R006 for active-set freshness
    behind the in-set distinction. Closes N6.

**Not edited:**
- `J001-O004-triage-speed/README.md` — RSK001 description left
  alone; the existing wording already justifies the
  RSK001 → R003 mapping (see N5). Risk-requirement map
  unchanged.

No Change Record — judge inside a create cycle (S006 / S008 /
S010 / S012 / S014 / S018 precedent, `workflows.md`).

## Coverage check

- Risk-requirement map intact: ✓ all 5 risks covered; all 5
  requirements mapped; map unchanged from S020. RSK001 → R001,
  R003 (composition rationale now lives on R003 Detail per N5).
- Per-doc dependency graph: still a connected DAG, no cycles.
  R001 and R004 remain intra-outcome hubs (zero intra-outcome
  deps; R004's snapshot framing keeps it acyclic with R003).
  R002 → R001; R003 → R001, R004; R005 → R001, R002. Cross-
  outcome edges grew from 7 to 10:
  - R002: O001-R001 (new), O001-R002, O003-R001, O003-R002,
    O003-R003 (new). Five edges; R002 is the list-level
    aggregator hub for cross-outcome signals.
  - R003: O003-R003.
  - R004: O002-R006, O003-R002.
  - R005: O002-R001, O002-R006 (new).
- Statement lines match outcome README verbatim: ✓ (all 5).
- All ACs solution-free (no UI nouns, no thresholds, no scale
  specifications): ✓ — no AC text changed this session; new R001
  Edge Case stays solution-free (names the shape, not the
  mechanism).
- All statements follow rubric pattern `[Product/Solution] must
  [Capability/Constraint]`: ✓.
- PDR citations consistent with PDR Decisions: ✓ unchanged.
  PRD001 cited in R001/R002/R003/R004/R005 Edge Cases; PRD002
  cited in R002 (two-timestamp shape), R004 ("what has
  changed"), R005 EC3 (stale dependency). PRD003 still not
  cited — no shared-capability requirement under O004.
- **RSK001 (No Urgency Ranking) — two mitigators (R001, R003).**
  Composition is honest: R001 supplies granular ranking, R003
  reduces the evaluation cost the risk names. Rationale now
  carried on R003 Detail. Not a fragility.

## Vertical & horizontal trace

- **Vertical:** every requirement traces to a mapped risk;
  every risk impact clause closes on the outcome statement
  ("identify which items most need intervention right now").
  Unchanged.
- **Sibling (intra-outcome) — sharpened this session:**
  - R001 (ranking hub) now acknowledges dep-blocked-items as
    an input (EC), closing the R001 ↔ R005 asymmetry.
  - R003 ↔ R004 enum-posture asymmetry now explicit on R003's
    side, with the workflow-vocabulary vs product-output
    distinction named.
  - R004 (memory hub) snapshot framing made explicit; keeps
    R004 acyclic with R003 by characterization rather than by
    silence.
- **Cousin (cross-outcome) — sharpened this session:**
  - **R002 cross-outcome edge set extended.** Now covers all
    four named signal categories from R002 Detail (slip via
    O001-R001 + O001-R002, status basis via O003-R001, status
    freshness via O003-R002, status confidence via O003-R003).
    Detail and Dependencies now in agreement.
  - **R005 → O002-R006 added.** "In-set" depends on a
    refreshed picture; same consumption pattern as R004 →
    O002-R006 (already declared).
  - **R003 ↔ O003-R004 (status override) — parallel pattern,
    not consumption.** Held; S020's "flagged for judge" prompt
    resolved as hold. PDR pressure not yet sufficient (two
    instances; PRD001/PRD002 each had 3+).
  - **No-owner-at-all seam between O002 and O005** — unchanged
    from S012/S017/S018/S019/S020; not touched.

## Deferred (so we don't lose it)

New this session:

- **R001 / R002 / R004 history capability** — R004 now
  explicitly records a snapshot per decision (rank, signals,
  recommendation, decision, freshness anchors). Third
  occurrence of audit/event-store pressure across outcomes
  after S014 (O001 R001/R003/R004) and S018 (O003
  R001/R002/R004). Engineering-layer. Sharpest version yet.
- **R002 cross-outcome dep breadth (now 5 edges)** — load-
  bearing because R002 is the list-level aggregator. If O005
  introduces another list-level signal, R002 may grow to 6 or
  7 edges. Worth watching as architecture pressure; not a
  PDR candidate (this is structural, not posture).
- **R003/R004 enum-posture asymmetry now named in-doc** —
  reviewer should probe whether the explanatory paragraph
  needs to be promoted (e.g., into a PDR alongside any future
  "override surfacing" PDR), or whether in-doc explanation is
  enough.

Promoted / sharpened PDR-candidates:

- **"Override surfacing" as cross-outcome pattern** — flagged
  in S020. Unchanged this session: two instances (O003-R004
  status override, O004-R003 recommendation override) is not
  yet enough pressure. Re-evaluate during O005 drafting when
  O005-R004 (Escalation Timing Recommendation) lands —
  README wording ("which the bird-dogger can adopt or
  override") suggests a third instance.

Carried forward (unchanged or sharpened):

- **PRD001 / PRD002 cited rather than re-derived.** Worked
  cleanly through the judge pass; no R-doc was dinged for
  re-stating posture, and PDR citations didn't surface any
  contradiction issues. Format validated. Same for the
  Edge-Cases-inline citation style.
- **PRD003** — still uncited in O004 (no shared-capability
  requirement). Carries forward to O005 evaluation.
- **R005 cycle-handling commitment** held as edge case
  (S020 prompt 4 resolved). Standing if reviewer pushes to
  lift.
- **Engineering concerns sharpened by this session:**
  - **Decisioning component** — unchanged shape; R003's closed
    enum is now an explicit ADR-shaped commitment.
  - **Per-item cache with freshness** — unchanged.
  - **History / event store** — sharpened by R004 snapshot
    framing. Now an explicit "this is captured and stored"
    rather than an implicit "we record."
  - **Dependency graph store** — unchanged from S020 (R005).
- **O001-RSK002 single-mitigation fragility.** Standing from
  S006. Unchanged.
- **No-owner-at-all seam between O002 and O005.** Standing
  from S012. Not touched.
- **Missing session 016 (O002 judge).** Standing user
  decision: do not recover, do not re-run. Unchanged.
- **Workspace path mismatch in Cursor.** Seven sessions
  running with the absolute-path workaround.

Cleared:

- **PDR-citation format precedent (S020)** — validated this
  session. Edge-Cases-inline format is the standing convention
  unless review changes it.
- **S020's six standing judge prompts** — all six resolved
  above (1 → hold, 2 → accept, 3 → accept-and-extend,
  4 → hold, 5 → hold, 6 → hold-but-related-edit).

## Recommended next step

1. **Commit + PR the five edited R-doc files on branch
   `product-docs`** so an independent reviewer can run session
   022 (review) against the Requirement rubric. Reviewer-
   fruitful probes:
   - R002 Dependencies breadth (now 5 cross-outcome edges) —
     accept as load-bearing list-level aggregation, or push
     for consolidation?
   - R001 new EC for dep-blocked items — does it commit
     enough (e.g., should "does not silently outrank" become
     an AC), or is EC the right home?
   - R003 closed-enum explanatory paragraph — accept in-doc,
     or push for a PDR codifying "product-output enums are
     closed; workflow-label enums are open"?
   - R004 snapshot-vs-live-read sentence — does "shape-of, not
     live-on" land, or does it need a defined term?
   - R005 → O002-R006 added — accept the freshness-behind-in-
     set framing, or push for further coupling between R005
     and the active-set lifecycle?
   - R003 cites RSK001 alongside RSK003 — accept, or push for
     the README to also acknowledge the composition?
   - Standing-hold reviewer probes (carried from S020):
     R003 override-shape vs O003-R004 (PDR or not), R005
     cycle commitment (lift or hold), R001 granularity floor
     (tighter or hold), R004 "what has changed" enumeration.
2. **Then draft O005 R-docs (Escalation Quality).** S020's
   recommended-next-step #2 still holds. Five requirements
   per the README: R001 Escalation Target, R002 Owner
   Currency, R003 Response Window Visibility, R004 Escalation
   Timing Recommendation, R005 Escalation Record. R004 is the
   high-priority cousin to O004-R003 (recommendation
   override) — the "override surfacing" PDR pressure
   re-evaluates during that drafting.
3. **PDR judge pass on PRD001/PRD002/PRD003** — explicitly
   off the table per S019 user decision; do not propose.
   Reviewer feedback on the PRs may merit changes; those go
   through update cycles with CRs scoped to the changed
   PDR(s).

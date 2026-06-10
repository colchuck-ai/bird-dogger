# Session 019 — Drafted three product-level PDRs

## Goal

Open the long-deferred PDR session for the three cross-outcome
PDR-candidates flagged across S014, S015, S017, and S018. User's
opening ask was "draft requirements for the next important
outcome" (O004 by upstream-completion logic); the standing
PDR-first deferral was now four sessions deep, so this session
surfaced the trade-off explicitly. User pivoted to PDR-first, then
chose all three PDRs together.

## Context loaded

- `docs/product/README.md` — product root; J001 + five outcomes.
- All five outcome READMEs (O001–O005) — for the risk-requirement
  maps and to confirm O004/O005 had judged outcome READMEs but no
  per-requirement R-docs yet.
- The 10 R-docs that exhibit the patterns the three PDRs codify:
  - "Cannot assess" / coverage-over-precision instances:
    O001-R001, O001-R002, O001-R004, O001-R006, O002-R001,
    O002-R004, O002-R006, O003-R002, O003-R005, O003-R006.
  - Per-item freshness instances: O001-R002, O002-R006 (set-level
    cousin), O003-R002.
  - Source registration duplicates: O001-R007, O002-R007.
- `intent` skill: `SKILL.md` (PDR rubric in Records section),
  `references/workflows.md` (create cycle has no CR; PDRs are
  decision records themselves), `references/context-tracing.md`
  (records load only when needed),
  `assets/templates/pdr-simple.md`.
- `resources/previous-sessions/017-requirements-o003-drafted.md`,
  `018-requirements-o003-judged.md` — most recent precedent;
  S018's recommended-next-step #2 was exactly this PDR session.

Skipped: did not load O004/O005 R-docs (none exist). Did not load
component or architecture docs (none exist). Did not load older
session digests (009/010 outcome judges) — S017 + S018 carried
forward the standing flags I needed.

## Workspace path mismatch (still standing from S015 onward)

Unchanged from S017/S018. Cursor opens
`/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`; real repo is
`/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. Used the
S017/S018 workaround: absolute paths through the real repo for
all writes; shell `pwd` was already on the real path so reads
worked. `git status` on `bird-dogger/` shows the three new PDRs
plus this digest as the only untracked changes from this session.

**Standing for next session:** repoint Cursor workspace at
`/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. Five
sessions running with this workaround now.

## Outcome selection — pivoted to PDR-first

User asked for the next outcome's R-docs. I surfaced the standing
trade-off via AskQuestion across three options (O004 now, O005
now, PDR-first). User selected PDR-first.

S017 had asked the same kind of question and the user chose to
push down into the next outcome instead; this session reversed
that by accepting the four-sessions-deep recommendation. The
mechanic (AskQuestion before drafting begins, when a load-bearing
sequencing call is open) is the same.

Then a second AskQuestion across three PDR-scope options (all
three together; PRD001 only; PRD002+PRD003 deferring PRD001).
User selected all three together — same as S017's
"fold-them-together" recommendation.

## Template choice

Simple template for all three. None of the three carry attached
material (no nested CRs, no nested ADRs, no follow-up artifacts
yet). Same precedent as the R-doc drafting sessions used the
simple template throughout.

## PDR framing decisions

Cross-cutting moves applied to all three:

- **Plain-language summary at the top.** Each PDR opens with a
  one-sentence statement of what was decided, then expands into
  Context / Options / Decision / Consequences per the simple
  template.
- **Context cites concrete R-docs by ID, not patterns.** Each
  PDR's Context section lists the specific R-docs it codifies.
  PRD001 names ten; PRD002 names three; PRD003 names two. This
  ties the PDR to the existing requirement layer rather than
  asserting a posture in the abstract.
- **Options name real alternatives that were under pressure.**
  Not strawmen. PRD001's Option B (precision over coverage) is
  the actual default a future judge could pick if the posture
  weren't recorded; PRD003's Option B (single-home requirement)
  is what S015 nearly proposed for source registration.
- **Consequences read through to specific downstream
  requirements and components, not in the abstract.** Each PDR's
  Consequences section names which future R-docs benefit, which
  engineering-layer concerns it constrains, and what
  follow-up work it implies.
- **Cross-PDR reference where it's load-bearing.** PRD002 cites
  PRD001 explicitly: when a source does not expose an
  underlying-data-change time, the data-change timestamp surfaces
  as "unknown" per PRD001. The two PDRs reinforce each other —
  coverage-over-precision is the posture; the two-timestamp
  shape is one place the posture is realized.

Per-PDR notes:

- **PRD001 (Coverage Over Precision).** The posture PDR. Lists
  ten concrete R-doc instances. Decision says coverage-over-
  precision composes across outcomes (status confidence feeds
  urgency feeds escalation timing) where mixed posture would
  break the chain. Consequences explicitly invert the judge
  prompt: pull on requirements that *do not* exhibit the
  posture, not requirements that do.
- **PRD002 (Per-Item Freshness Presentation).** Two-timestamp
  per-item shape ("last assessed" + "last underlying data
  change"). Set-level freshness (O002-R006) named as a cousin
  shape under the same posture, not collapsed into per-item.
  Multi-source picking rule explicitly punted to ADR territory
  — same engineering-layer punt O003-R002 already made; this
  PDR ratifies it.
- **PRD003 (Shared Source Registration).** Duplicate at
  requirement layer, share at component layer. Decision
  preserves per-outcome trace (O001-RSK005 and O002-RSK002 each
  map to a requirement under their own outcome). Consequences
  flag the drift risk explicitly — without a parallel-edit
  prompt in the judge rubric, drift between R007s is a real
  cost. Also flags the future de-registration capability
  (currently in O001-R006 EC3) as following the same pattern.

## User decision: no judge session on the three PDRs

Recorded explicitly per user this session. The PDR session is
done; the next step is review (commit + PR + reviewer), not
judge.

**Consequences of that decision — load-bearing for what
follows:**

- **Reviewer is the only quality gate before merge.** The
  assistant has not pre-checked the three PDRs against the PDR
  rubric in `intent/SKILL.md`. Anything a judge would have
  surfaced (rubric-correctness, options completeness,
  consequence honesty, cross-PDR coherence) is now on the
  reviewer.
- **All "boundary calls worth flagging for the judge" type
  flags below are reviewer flags, not held-back judge work.**
  Same shape as S017's missing-016 framing applied to a different
  point in the cycle.
- **Future sessions should not propose recovering the judge
  pass on these PDRs.** Same pattern as S017's "do not propose
  reconstructing 016" rule. If a later session needs to know
  "what would the PDR judge have caught," diff the drafted state
  against any reviewer feedback that lands in the PR.

## Boundary calls worth flagging for the reviewer

(Not held-back judge work; flagged because user explicitly
declined the judge step and the reviewer is the only remaining
quality gate.)

- **PRD001 generality across O004 / O005.** Coverage over
  precision is named as composing across outcomes. O005's
  escalation timing has a real edge case where the bird-dogger
  asks "what's the best guess on when to escalate?" — a
  precision-first read on a specific question rather than the
  posture. PRD001's Consequences allow per-requirement deviation
  with explicit justification, but the reviewer should probe
  whether "best guess on timing" is actually a deviation or
  whether O005 wants a different posture entirely.
- **PRD002 set-level scope.** PRD002 explicitly excludes
  set-level freshness (O002-R006) from its Decision and names it
  as a "separate cousin shape under the same posture." Reviewer
  may push for a single PDR covering both scopes, or push for a
  separate cousin PDR codifying the set-level shape. Either
  direction is defensible; the current scope is the smaller
  choice.
- **PRD002 ↔ PRD001 cross-reference direction.** PRD002 cites
  PRD001 (forward-only). PRD001 does not cite PRD002 back.
  Justification: PRD001 is the broader posture; PRD002 is one
  realization. Reviewer may push for a back-reference for
  symmetry.
- **PRD003 drift-control mechanism.** Consequences flag the
  drift risk between O001-R007 and O002-R007 but defer the
  enforcement to "the next judge rubric pass should add a
  standing prompt." Without that prompt landing, the drift risk
  is named but not addressed. Reviewer may push for a more
  concrete enforcement (e.g., a CR-on-either-requires-CR-on-both
  rule recorded somewhere).
- **PRD003 future de-registration analogue.** PRD003 names
  de-registration as following the same pattern when drafted.
  De-registration is currently a single edge case in O001-R006
  EC3, not a requirement. Reviewer may push: is naming a future
  analogue scope-creep for this PDR, or is it useful trajectory?
- **No `Decision` rubric for "options balance."** The PDR
  template doesn't require options to be balanced (some real,
  some strawmen), only that alternatives were considered. All
  three PDRs lean somewhat toward the chosen option in framing.
  Reviewer may push for sharper steelmanning of options B/C in
  any of the three.

## What got written

Three files under `docs/product/pdrs/` (new directory):

- `PRD001-coverage-over-precision.md`
- `PRD002-per-item-freshness-presentation.md`
- `PRD003-shared-source-registration.md`

All simple template. Each: one-sentence summary, Context (with
concrete R-doc citations), Options (three alternatives), Decision
(rationale grounded in J001 outcome framing), Consequences
(positive + negative + follow-up).

No Change Record — drafting in a create cycle (S005 / S007 /
S009 / S011 / S013 / S015 / S017 precedent extended to PDRs).

## Coverage check

- All three PDR-candidates from S017/S018 are now drafted: ✓.
- Each PDR cites at least one R-doc by ID in its Context: ✓ (10,
  3, 2 respectively).
- Each PDR's Decision is unambiguous (single named option, not
  hedged): ✓.
- Each PDR's Consequences include both positive and negative or
  follow-up: ✓ (template-required).
- No PDR contradicts existing R-doc text. PDRs codify what is
  already there; they do not introduce new commitments.
- File-naming convention matches the SKILL.md `Files` section:
  product-level path `docs/product/pdrs/PRD<NNN>-<name>.md`: ✓.
- PRD numbering starts at 001: ✓ (no prior PDRs in repo).

## Vertical & horizontal trace

- **Vertical (per PDR):**
  - PRD001 traces to all five outcomes via the ten cited R-docs;
    the posture composes across outcomes per its Decision.
  - PRD002 traces to O001-R002, O003-R002 (per-item) and
    O002-R006 (set-level cousin); reads through to O004 R002 +
    R004 (prospective consumers) and O003-R005 (existing
    consumer of the gap signal).
  - PRD003 traces to O001-R007, O002-R007 and reads through to
    the risk-requirement maps (O001-RSK005, O002-RSK002).
- **Horizontal (between PDRs):**
  - PRD001 ↔ PRD002 — load-bearing forward reference from PRD002
    to PRD001 ("unknown data-update time surfaces per PRD001").
    Reviewer flag noted above.
  - PRD001 ↔ PRD003 — no load-bearing edge. PRD003's "no silent
    swallow" framing for unsupported source types is consistent
    with PRD001's posture but not cited; the connection is
    light enough to leave implicit.
  - PRD002 ↔ PRD003 — no edge. Independent decisions on
    independent topics.

## Deferred (so we don't lose it)

New this session:

- **Reviewer becomes the only quality gate on the PDRs.** User
  declined judge. All boundary-call flags above are reviewer
  flags, not held-back judge work. Future sessions should not
  propose recovering the judge pass.
- **De-registration capability** — flagged in PRD003
  Consequences as following the shared-capability pattern when
  drafted. Currently a single edge case in O001-R006 EC3, not a
  requirement. Promote when O002 (or a future outcome) raises a
  concrete need.

Cleared this session:

- **PDR-first deferral** (standing from S014 / S015 / S017 /
  S018). Three PDR-candidates drafted; deferral closed. O004 /
  O005 R-docs can now reference the PDRs rather than re-deriving
  the posture, the freshness shape, or the source-registration
  duplication.

Carried forward (unchanged):

- **Workspace path mismatch in Cursor.** Five sessions running
  with the absolute-path workaround. Repoint at the real repo to
  end the cost.
- **Missing session 016 (O002 judge).** Standing user decision:
  do not recover, do not re-run. Method for reading 016's
  findings unchanged: diff S015's described draft state against
  the current O002 R-docs.
- **O001-RSK002 single-mitigation fragility.** Standing from
  S006. Not touched.
- **No-owner-at-all seam between O002 and O005.** Standing from
  S012. Not touched.
- **R001/R002/R004 history capability** (audit/event store
  pressure across O001 + O003). Standing from S014 + S018.
  Engineering-layer; not addressable at PDR layer.
- **Override is a four-requirement shape (O003 R001/R002/R004/
  R005/R006).** Standing from S017/S018. Component-boundary
  flag for the engineering layer.
- **Engineering concerns sharpened by this session:**
  - **Decisioning component** — pressure unchanged.
  - **Per-item cache with freshness** — PRD002 ratifies the
    two-timestamp per-item shape; engineering-layer cache must
    carry both timestamps distinctly.
  - **History / event store** — pressure unchanged.
  - **Source Registration component** — PRD003 commits to one
    component serving both O001-R007 and O002-R007. ADR
    territory when engineering layer lands.

## Recommended next step

1. **Commit + PR the three PDRs on branch `product-docs`** so an
   independent reviewer can run a review session. The reviewer
   is the only quality gate before merge per user's no-judge
   decision; their probes should focus on the boundary calls
   listed above (PRD001 generality across O005, PRD002 scope
   choice, PRD002 ↔ PRD001 reference symmetry, PRD003
   drift-control concreteness, PRD003 de-registration trajectory,
   options balance across all three).
2. **Then draft O004 R-docs** (Triage Speed). Five requirements:
   R001 Urgency Ranking, R002 List-Level Urgency Signals, R003
   Intervention Recommendation, R004 Triage State Carry-Forward,
   R005 Inter-Item Dependency Surfacing. Per-requirement docs
   should be smaller than O001/O002/O003 R-docs because Edge
   Cases that previously re-derived "cannot assess" can
   reference PRD001, freshness presentation can reference
   PRD002, and shared-capability framing can reference PRD003.
3. **Then O005 R-docs** (Escalation Quality). Furthest
   downstream; consumes O001 + O003 + O004. Drafting after O004
   completes the requirement layer for J001.
4. **PDR judge pass — explicitly off the table per user
   decision this session.** Do not propose. If reviewer feedback
   lands and merits changes, those go through update cycles
   with CRs scoped to the changed PDR(s), not a retroactive
   judge.

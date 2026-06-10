# Session 017 — Drafted requirement documents for O003 Status Fidelity

## Missing session 016

A judge session on the O002 requirement docs (the natural session
016 in this sequence) was run by the user out-of-band and is **not
captured in `resources/previous-sessions/`**. No digest exists.
S015's recommended-next-step #2 named exactly this judge session as
the right next move; that work happened, but its findings, hold
decisions, and any prunes to the O002 dep graph are not written
down here.

**Standing for next session:** before judging O003 (or opening the
PDR session in S015's deferred list), reconstruct or re-run the
O002 judge so its findings are durable. Several O002 standing flags
named in S015 ("R001 ↔ R003 cycle," "R001 ↔ R007 cycle," "R002 ↔
R005 cycle," "R003 rejected-candidates vs. O004-R004," "R008 de-
registration gap") may already be resolved in the missing 016 —
this session cannot tell.

This session did not consult the missing 016 (no artifact to read)
and proceeded as if O002 were drafted-but-not-judged.

## Goal

Use the `intent` skill to draft the per-requirement documents for
the next important outcome under J001. S015's recommended-next-step
strongly preferred PDR-first; user chose to push down into the next
outcome instead. The PDR-first deferral is now standing for the
third session in a row.

## Context loaded

- `docs/product/README.md` — product root; J001 + five outcomes.
- `docs/product/outcomes/J001-O003-status-fidelity/README.md` —
  outcome under drafting; requirement statements lifted verbatim;
  risk-requirement map untouched.
- `docs/product/outcomes/J001-O001-slip-detection-lag/README.md`
  and `J001-O002-coverage/README.md` — sibling outcomes already
  drafted; R002 parallel-freshness pattern (O001-R002, O003-R002)
  and R006 refresh trigger (O002-R006) named in O003 cross-outcome
  deps.
- `docs/product/outcomes/J001-O004-triage-speed/README.md`,
  `J001-O005-escalation-quality/README.md` — downstream consumers;
  read for what they expect from status (urgency signals,
  escalation timing) but not internalized in O003 docs.
- Two sample requirement files —
  `J001-O001-R001-proactive-slip-surfacing.md` and
  `J001-O002-R001-coverage-set-visibility.md` — for in-doc style
  + verbatim-statement precedent.
- `resources/previous-sessions/013-requirements-o001-drafted.md`,
  `014-requirements-o001-judged.md`,
  `015-requirements-o002-drafted.md` — drafting + judge precedent.
- `intent` skill: `SKILL.md` (Requirement rubric), template
  `requirement-simple.md`. Did not re-read `context-tracing.md` or
  `workflows.md` in full this session; relied on S013/S015 codified
  decisions for create-cycle behavior (no CR, no Records).

Skipped: Records (CRs, PDRs, ADRs) per create-cycle precedent. Did
not load O001-R002 or O002-R006 in full — only their statement
lines named in cross-outcome deps.

## Workspace path mismatch — still standing, worked around

S015's note holds: Cursor workspace points at
`jomadu/bird-dog-skill/` (sparse, no `.git`), the real repo is
`colchuck-ai/bird-dogger/`. This session:

- Read tool calls against the workspace path failed; fell back to
  shell-relative reads which worked because shell `pwd` was the
  colchuck path.
- Write tool calls landed correctly in `colchuck-ai/bird-dogger/`
  by passing absolute paths to Write — different workaround than
  S015's "write then cp." Cleaner; no relocation step needed.
- `git status` on `bird-dogger/` shows the 6 R-docs + (this
  digest) as the only untracked changes from this session.

**Standing for next session:** unchanged from S015 — repoint the
Cursor workspace at `/Users/max.dunn/dev/personal/colchuck-ai/
bird-dogger`. Until then, future sessions must remember to use
absolute paths on writes or replay the cp dance.

## Outcome selection

User asked for "the next important outcome." Pre-survey against
S015's coupling lens (already done; not re-derived):

- **O003 Status Fidelity** — feeds both O004 (urgency from status)
  and O005 (escalation timing from status); last upstream piece
  before downstream outcomes become well-grounded; R002 is a
  direct parallel of O001-R002, which makes the per-item freshness
  PDR pressure visible at draft time rather than after.
- **O004 Triage Speed** — consumes O001 + O002 + O003; drafting
  it without O003 forces guesses about what status signals it can
  rely on.
- **O005 Escalation Quality** — consumes O001 + O004 + O003;
  furthest downstream.

Recommended **O003** for upstream completion + per-item freshness
PDR pressure visibility. User confirmed via AskQuestion across
five options (O003, PDR-first, judge-O002-first, O004, O005). The
PDR-first and judge-first options were offered explicitly per
S015's recommendation; user chose draft anyway.

## Template choice

Simple template for all 6. None of the O003 requirements carry a
PDR, CR, or other attached material yet. Same precedent as O001
(S013) and O002 (S015).

## Requirement framing decisions

Statement lines lifted verbatim from the outcome README's
requirements list — no rewording. Same trace-preservation pattern
as S013 / S015.

Cross-cutting moves applied to all 6:

- **"Cannot assess" / coverage-over-precision posture used
  implicitly.** R002 (unknown data-update time surfaced
  explicitly), R005 (deferred re-assessment surfaced as stale,
  not silently held), R006 (single-source items not flagged as
  conflicting; conflicts not silently resolved). Same posture
  S014/S015 flagged as PDR-candidate. Now used a third time
  without the PDR landing.
- **Acceptance criteria stay solution-free.** No UI nouns, no
  channel choices, no thresholds, no scale specifications. S013/
  S015 precedent.
- **Edge cases bias toward fail-safe defaults.** Every doc
  carries at least one edge case where "we don't know" or "the
  evidence disagrees" forces explicit surfacing rather than
  silent omission (R001 missing source named in basis, R002
  unknown data-update time, R003 sparse signal forces low
  confidence not default high, R004 override conflict re-surfaced
  as candidate, R005 unavailable-source defer + stale surfacing,
  R006 override-vs-source surfaced not aligned).
- **R001 explicitly framed as the basis hub** with no intra-
  outcome dependencies; R003, R004, R006 attach to it. R002 also
  has no intra-outcome deps. Avoids the R001-as-hub cycle problem
  S015 hit in O002 (R001 ↔ R003 ↔ R007). No intra-outcome cycles
  in O003 — see Coverage check.

Per-requirement notes:

- **R001 (Status Basis).** Hub for the inputs + reasoning trace.
  Names overrides (R004) as first-class in the basis to head off
  "override is a separate mode" framing. Cross-outcome edge to
  O001-R006 named only in the edge case (per-item source-
  availability), not in deps — same restraint as S015 used.
- **R002 (Status Freshness).** Two-timestamp framing is the
  central commitment: "assessed" and "underlying data" distinct,
  not collapsed. **Cross-outcome dep on O001-R002 named
  explicitly as a parallel pattern flag**, not a strict
  consumption dep — same restraint S015 wished for in O002-R007
  but didn't apply. Per-item freshness now in three docs across
  two outcomes (O001-R002, O002-R006 refresh-time variant,
  O003-R002).
- **R003 (Status Confidence).** "Differentiates among levels,
  not binary" without prescribing scale — same shape as O001-R005
  silence-window held by S014 judge. Pattern continues.
  Override (R004) explicitly excluded from inferred confidence —
  AC names this.
- **R004 (Status Override).** Override-survives-reassessment is
  the central commitment. Re-assessed result under override
  framed as a "candidate alongside, treated as conflict (R006)"
  — mildly prescriptive at the requirement layer; the word
  "candidate" introduces a status-object subtype not named
  anywhere else. Judge will probe.
- **R005 (Re-Assessment On Change).** Detection mechanism (poll
  vs. push vs. hash) explicitly engineering-layer. Cross-outcome
  dep on O002-R006 named (refresh as the upstream signal that
  surfaces newly-changed data). High-frequency churn edge case
  asserts "assessment not older than latest change at point of
  surfacing" — verifiable without specifying interval.
- **R006 (Conflict Surfacing).** "Honesty about disagreement,
  not resolution" is the central commitment. **Introduces an
  "override-vs-source" conflict subtype** in the edge cases — a
  silent expansion of what counts as a conflict beyond
  multi-source disagreement. Judge will probe whether this is
  R006 or R004.

## Boundary calls worth flagging for the judge

- **R002 parallel-pattern dep on O001-R002.** Listed in deps with
  the qualifier "parallel per-item freshness pattern under a
  different outcome; flagged for PDR consolidation." Honest about
  the relationship; PDR pressure now explicit in three R-docs.
- **R004 "candidate alongside override" language.** Introduces a
  status-object subtype not named in any other requirement.
  Either lift it into R001 basis (override + candidate are both
  basis facets) or accept the local naming. Mildly prescriptive.
- **R006 "override-vs-source" conflict subtype.** R006's scope as
  literally stated in the outcome README is "sources disagree" —
  override vs. sources is a different shape. Either R006 widens
  to include it (this session's read) or R004 absorbs it.
- **R003 confidence scale unspecified.** "Differentiates among
  levels" without naming categorical vs. numeric or how many
  levels. Same shape as O001-R005 silence-window held by S014
  judge. Likely held.
- **R005 detection mechanism deferred to engineering.** Same
  shape as O002-R006 refresh-trigger deferral S015 flagged.
  Reviewer may probe.
- **R005 "before its status is next surfaced" timing.** The
  re-assessment timing commitment is bound to checkpoint
  surfacing rather than to a clock interval. Verifiable but
  binds R005 tightly to the checkpoint cadence assumption.

## What got written

Six files under
`docs/product/outcomes/J001-O003-status-fidelity/requirements/`:

- `J001-O003-R001-status-basis.md`
- `J001-O003-R002-status-freshness.md`
- `J001-O003-R003-status-confidence.md`
- `J001-O003-R004-status-override.md`
- `J001-O003-R005-re-assessment-on-change.md`
- `J001-O003-R006-conflict-surfacing.md`

All simple template. Each: lifted statement, Detail, Edge Cases,
Examples (one scenario each), Acceptance Criteria (3 conditions
each), Dependencies (intra-outcome plus the two named
cross-outcome edges on R002 → O001-R002, R005 → O002-R006).

No Change Record — drafting session in a create cycle (S005 /
S007 / S009 / S011 / S013 / S015 precedent).

## Coverage check

- Every requirement listed in the O003 README has a per-
  requirement document: ✓ (6 of 6).
- Every per-requirement document's statement matches the outcome
  README verbatim: ✓ (trace-preserving).
- Dependency graph: connected, **no cycles**. R001 and R002 have
  no intra-outcome deps (hubs); R003 → R001; R004 → R001, R005;
  R005 → R002; R006 → R001, R003. Improvement over O002's three-
  cycle draft state (S015) — O003 was drafted with the O002
  cycle lesson in hand.
- Cross-outcome dep edges asserted (R002 → O001-R002 parallel,
  R005 → O002-R006 upstream) are consistent with the named
  outcomes' drafted state.
- Risk–requirement map in the O003 README is unchanged and
  intact: RSK001 → R001, R003, R004; RSK002 → R002, R005;
  RSK003 → R001, R004; RSK004 → R001, R006.

## Vertical & horizontal trace

- **Vertical:** every requirement traces to a mapped risk; every
  risk impact clause closes on the outcome statement ("tracked
  status reflects its true status").
- **Sibling (intra-outcome):** R001 is the basis hub; R002 is
  the freshness hub; both stand alone. R003, R004, R006 attach
  to the basis hub; R005 attaches to the freshness hub via R002.
  Acyclic by construction.
- **Cousin (cross-outcome) — sharpened this session:**
  - **O003-R002 parallel to O001-R002.** Same two-timestamp
    presentation pattern under a different outcome. PDR-candidate
    pressure now explicit in three R-docs across two outcomes
    (the third being O002-R006's refresh-time timestamp).
  - **O003-R005 consumes O002-R006.** Active-set refresh
    surfaces newly-changed source data; R005 is the per-item
    reassessment loop on top of that refresh. Real consumption
    dep, not adjacency.
  - **O003-R006 feeds O001-R001.** When source disagreement
    happens on slip-relevant signals, R001's "cannot assess"
    state is the natural downstream surfacing. Not asserted in
    deps (R006 doesn't know what's slip-relevant) but worth
    flagging.
  - **O003-R004 ↔ O002-R002 (Manual Item Capture).** Manual
    items can only carry an overridden status. Named in R004's
    edge case, not in deps.
  - **No-owner-at-all seam between O002 and O005** — unchanged
    from S012/S015; not touched by O003.

## Deferred (so we don't lose it)

New this session:

- **R004 "candidate alongside override" subtype.** Status-object
  vocabulary not named in any other requirement.
- **R006 "override-vs-source" conflict subtype.** Widens R006
  scope beyond the literal README statement.
- **R005 timing bound to checkpoint cadence** rather than clock
  interval. Verifiable but couples R005 to a checkpoint
  assumption not formalized elsewhere.
- **Missing session 016 (O002 judge).** Findings exist in the
  user's head; not in the repo. Recover or re-run before
  judging O003.

Promoted / sharpened PDR-candidates (deferred since S014;
pressure increased again this session):

- **"Cannot assess" / coverage-over-precision posture.** Now
  used in **eight** per-requirement docs across three outcomes
  (O001-R001/R004/R006, O002-R001/R004/R006, O003-R002/R005/
  R006). Each new outcome repeats the pattern. PDR needed
  before O004/O005 to stop the accumulation.
- **Per-item freshness presentation ("last assessed" + "last
  underlying change") as a cross-outcome capability.** Three
  docs now (O001-R002, O002-R006 refresh variant, O003-R002).
  PDR worth opening jointly with the "cannot assess" PDR — same
  family of per-item presentation contracts.
- **R007 Source Registration as cross-outcome capability**
  (carried from S015). Two literally-duplicated docs (O001-R007,
  O002-R007). Unchanged this session.

Carried forward:

- **All O002 standing flags from S015** ("R001 ↔ R003 cycle,"
  "R001 ↔ R007 cycle," "R002 ↔ R005 cycle," "R003 rejected-
  candidates vs. O004-R004," "R008 de-registration gap," "R004
  'in one action' softness," "R005 ambiguous-migration
  verifiability," "R006 refresh trigger") — possibly resolved
  in the missing 016, not knowable from this session.
- **O001-RSK002 single-mitigation fragility.** Standing from
  S006. Unchanged.
- **No-owner-at-all seam between O002 and O005.** Standing from
  S012; not touched.
- **Engineering concerns** (sharpened):
  - **Decisioning component** — R005 detection mechanism, R006
    conflict surfacing logic join the existing decisioning-
    component pressure (O001-R001/R005, O002-R001/R005,
    O004-R001/R003, O005-R004).
  - **Per-item cache with freshness** — R002 two-timestamp
    presentation + R005 reassessment loop extend the per-item
    cache pattern S013/S014/S015 flagged.
  - **History / event store** — R001 basis history (override
    recoverable after clear), R004 cleared-override history
    extend the audit/event store pressure S014/S015 flagged.
  - **Override is a first-class system object** — R004 +
    R001 (basis includes override) + R005 (reassessment must
    respect override) + R006 (override-vs-source conflict)
    make override a four-requirement shape. Likely a component
    boundary in engineering.
- **Workspace path mismatch** in Cursor. Unchanged from S015.

Cleared:

- (none)

## Recommended next step

1. **Recover or re-run session 016 (O002 judge).** Without it,
   any judge pass on O003 will rediscover findings that may
   already exist; and the O002 R-docs may carry standing flags
   that have already been resolved. This is the most expensive
   thing to leave undone right now.
2. **Open the PDR session.** S014 → S015 → S017 have each
   recommended this; the pressure has tripled. Two PDRs worth
   opening together:
   - "Cannot assess" / coverage-over-precision posture (now
     eight per-requirement repetitions).
   - Per-item freshness presentation as a cross-outcome
     capability (three repetitions).
   Likely worth folding in R007 Source Registration as a third
   PDR in the same session since the cross-outcome-capability
   shape is the common theme.
3. **Judge session on the six O003 requirement docs** (after 1
   and 2). Likely-fruitful prompts (in priority order):
   - R002 ↔ O001-R002 parallel-pattern dep — accept the
     declared-as-parallel framing or push it harder.
   - R004 "candidate alongside override" subtype — lift to R001
     or accept local.
   - R006 "override-vs-source" conflict scope — widen R006 or
     absorb into R004.
   - R003 confidence scale — held or named?
   - R005 timing bound to checkpoint cadence — adequate or too
     tightly coupled?
   - R005 detection mechanism — engineering or requirement?
4. **Then decide on O004 / O005 drafting.** If the PDRs land,
   their per-requirement docs will be substantially smaller (no
   repeated posture, no repeated freshness shape, no repeated
   registration paragraph). If the PDRs don't land first, the
   repetition pressure will be at four outcomes deep by the end
   of O004 — at which point retroactive PDR application becomes
   a large rework pass.

# Session 018 — Judged requirement documents for O003 Status Fidelity

## Goal

Run the judge pass on the six per-requirement documents under
`docs/product/outcomes/J001-O003-status-fidelity/requirements/`
(drafted in S017) against the Requirement rubric in the `intent`
skill, before going to review. Second judge session below the
outcome layer; S014 was the first (O001 requirements).

## Context loaded

- All 6 R-docs under
  `docs/product/outcomes/J001-O003-status-fidelity/requirements/`
  — the drafts under judgment.
- `docs/product/outcomes/J001-O003-status-fidelity/README.md` —
  owning outcome; risk-requirement map for coverage check.
- `docs/product/README.md` — product root for vertical trace.
- `docs/product/outcomes/J001-O001-slip-detection-lag/README.md`
  — sibling; R002 parallel-pattern check (O003-R002 vs O001-R002).
- `docs/product/outcomes/J001-O002-coverage/README.md` — sibling;
  R006 upstream consumption check (O003-R005 → O002-R006).
- `docs/product/outcomes/J001-O004-triage-speed/README.md`,
  `J001-O005-escalation-quality/README.md` — sibling cousins;
  downstream consumers of status signals (read for fit, not
  internalized in O003 edits).
- Cousin requirement docs for declared cross-outcome edges:
  - `J001-O001-R002` (assessment freshness) — R002 parallel-pattern
    cousin.
  - `J001-O001-R006` (source availability) — referenced in R001 EC3
    and R005 EC1.
  - `J001-O002-R002` (manual item capture) — referenced in R004 EC3.
  - `J001-O002-R006` (active-set freshness) — upstream of R005.
- `resources/previous-sessions/014-requirements-o001-judged.md` —
  the only prior judge precedent below the outcome layer; standing
  prompts, soft-edit patterns, fold-vs-split resolutions.
- `resources/previous-sessions/015-requirements-o002-drafted.md`,
  `017-requirements-o003-drafted.md` — drafting precedent and the
  six standing judge prompts S017 explicitly flagged for this
  session. Missing-S016 user decision still standing (no recovery,
  no re-run).
- `intent` skill: `SKILL.md` (Requirement rubric),
  `references/context-tracing.md` (Requirement self/siblings/
  cousins), `references/workflows.md` (judge inside create cycle
  has no CR), `assets/templates/requirement-simple.md`.

Skipped: Records (CRs, PDRs, ADRs) per workflows guidance — judge
on a create cycle. Sibling-outcome READMEs (O004, O005) read for
trace fit but not for cousin requirement detail — neither has
drafted requirements yet.

## Workspace path mismatch (still standing from sessions 002–017)

Unchanged from S017. Cursor opens
`/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`; real repo is
`/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. Read tool
failed against the workspace path; `realpath` revealed the
mismatch on the first read attempt. Switched to absolute paths
through the real repo for all reads and writes — same workaround
S017 used (no `cp` dance needed). `git status` on `bird-dogger/`
shows the four edited files on branch `product-docs` (already
ahead of `origin/product-docs` and tracking it).

**Standing for next session:** repoint Cursor workspace at
`/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. Until
then, every session pays the same first-read cost.

## Verdict

Closer to S014's shape than S012's clean pass. Three clear soft
edits (R004 dep drop, R002 multi-source clause, README map fix),
two load-bearing scope calls (override-vs-source home, R004
"candidate" language) — both surfaced to the user via
AskQuestion, both resolved on the recommended path. Six total
edits across four files. No defects; no held-back fragility
beyond what S017 had already flagged.

## Standing judge prompts from S017 — resolutions

1. **R002 ↔ O001-R002 parallel-pattern dep — pass.** R002 names
   the relationship explicitly as "parallel per-item freshness
   pattern under a different outcome; flagged for PDR
   consolidation." Honest about the relationship without claiming
   consumption. Same restraint S015 wished for in O002-R007 but
   didn't apply. Same precedent as O001-R002's revised boundary
   clause (S014 prompt 6). No edit on R002 for this prompt.

2. **R004 "candidate alongside override" subtype — soft edit
   (tighten_local).** Load-bearing decision; user-selected.
   Replaced "candidate alongside, treated as a conflict (R006)"
   with "surfaced alongside the override for the bird-dogger to
   adopt or dismiss; the override is not silently replaced."
   Drops the coined subtype, drops the implicit R006 dep, keeps
   the protection commitment.

3. **R006 "override-vs-source" conflict scope — soft edit
   (move_to_r004).** Load-bearing decision; user-selected. R006
   Detail ¶3 and Edge case 3 both reframed to explicitly defer
   override-vs-source surfacing to R004. R004 Detail ¶3 broadened
   to absorb the surfacing commitment, with new AC #4. R006 stays
   scoped to source-vs-source as the README literally says. The
   absorption keeps R004 as the single home for override-related
   behavior.

4. **R003 confidence scale unspecified — hold.** "Differentiates
   among levels rather than only flagging 'confident' vs 'not'"
   parameterizes on scale. Same pattern as O001-R005
   silence-window held by S014 prompt 1. AC #2 is verifiable once
   a scale is committed elsewhere. No edit.

5. **R005 timing bound to checkpoint cadence — hold.**
   "Checkpoint" is product-root vocabulary defined in the product
   README's J001 intro ("a recurring moment when the bird-dogger
   reviews the active set..."). R005 keying off "before its
   status is next surfaced for a checkpoint" is consistent with
   the whole product framing, not an arbitrary external coupling.
   No edit.

6. **R005 detection mechanism deferred to engineering — hold.**
   Same shape as O002-R006 refresh-trigger deferral. Edge case 3
   ("not per event, only such that the assessment is not older
   than the latest change at the point of surfacing") gives a
   verifiable shape without picking poll/push/hash. No edit.

## New issues the judge surfaced

**N1. R004 → R005 dependency reads backward — soft edit
applied.** R004's Dependencies entry "J001-O003-R005 —
re-assessment must respect an active override rather than
overwrite it" articulates a constraint R004 *imposes on* R005,
not a consumption R004 *needs from* R005. R005's AC #2
self-mandates the override-respect behavior. Same precedent as
S014 prompt 2 (R001 dropped R002 because R002 self-mandated its
display). Dropped R005 entry from R004 Dependencies; R004 now
depends only on R001 (basis trace) cleanly.

**N2. R002 multi-source "underlying data" timestamp
underspecified — soft edit applied.** R002 says "when its
underlying data was last updated." For multi-source items (which
R006 confirms are in scope), the per-item data-update timestamp
could be max-of, per-source, or composite. Edge cases covered
absence (source doesn't expose update time) and never-assessed
but not the multi-source case. Extended R002 Detail's "does not
prescribe" clause with the multi-source case; the picking rule
is engineering-layer; R002 commits only to surfacing a single
per-item value.

**N3. Outcome README risk-requirement map missing R003 → RSK004
— soft edit applied.** R003's edge case explicitly says
"Sources disagree (R006): confidence reflects the conflict
(low)." That's R003 actively mitigating RSK004 (Conflicting
Signals) by ensuring conflict-driven uncertainty isn't laundered
into a confident reading. The README map only listed R001 + R006
against RSK004. Added R003 — map now matches the doc-level
mitigation claim.

**N4. R001 / R002 / R004 history capability — flag, not edit.**
Basis recoverable after override clear (R001 EC1, R004 EC2) plus
override timestamp distinct from data-update timestamp (R002
EC3) jointly assert an audit/event-store capability with no
owning requirement. Same shape as S014's R001/R003/R004 history
flag. Engineering-layer concern. Carried forward.

**N5. R006 dep on R003 reads as downstream-effect, not
consumption — flag, not edit.** "R003 — conflict lowers
inferred confidence" reads as "R006 produces a side effect on
R003" rather than "R006 consumes R003." Same loose dep
convention noted in S014/S017. Reviewer-fruitful, not
judge-fruitful. No edit.

## Load-bearing decisions (user-surfaced via AskQuestion)

Two genuine shape calls. Both surfaced explicitly to the user
before writing; both resolved on the recommended path:

- **override_conflict_home → move_to_r004.** Override-vs-source
  surfacing now lives on R004 (which already absorbs
  override-survives-reassessment). R006 stays scoped to
  source-vs-source per the README's literal statement. Cleaner
  separation of concerns; no R006 widening, no implicit R004
  dep.
- **r004_candidate_language → tighten_local.** R004's edge case
  1 dropped "candidate" subtype and "treated as a conflict
  (R006)" framing. Replaced with descriptive "surfaced alongside
  the override for the bird-dogger to adopt or dismiss." Drops
  the coined vocabulary; keeps the protection commitment.

S014 applied soft edits without asking; this session asked
because both calls had load-bearing scope-boundary
consequences (R004 absorbing R006's edge case ≠ tightening a
phrase). Same workflow rule, different threshold call.

## What got edited

Six edits, four files. Bullet form:

- `J001-O003-status-fidelity/README.md`
  - Risk-Requirement Map: added R003 to RSK004's mitigators
    (was R001, R006; now R001, R003, R006). Map-correctness fix;
    R003's edge case already claimed this role.
- `J001-O003-R002-status-freshness.md`
  - Detail: extended the "does not prescribe" clause with the
    multi-source per-item data-update timestamp case
    (engineering-layer picking rule; R002 commits only to a
    single per-item value).
- `J001-O003-R004-status-override.md`
  - Detail (¶3): broadened from "re-assessment must not silently
    replace" to also assert override-vs-source disagreement is
    surfaced alongside the override — explicitly R004's scope,
    not R006's.
  - Edge case 1: tightened — dropped "candidate" subtype and
    "treated as a conflict (R006)" framing; replaced with
    descriptive surfacing language.
  - Acceptance Criteria: added AC #4 covering the broadened
    surfacing commitment.
  - Dependencies: dropped R005 entry (S014 prompt 2 precedent;
    R005's AC #2 self-mandates).
- `J001-O003-R006-conflict-surfacing.md`
  - Detail (¶3): reframed — override-vs-source surfacing
    explicitly out of R006's scope, owned by R004.
  - Edge case 3: reframed as scope-boundary statement (was
    "surfaced as conflict (override-vs-source)"; now "out of
    R006's scope; R004 owns it").

No Change Record — judge inside a create cycle (S006 / S008 /
S010 / S012 / S014 precedent, `workflows.md`).

## Coverage check

- Risk-requirement map intact and sharpened: ✓ all 4 risks
  covered; all 6 requirements mapped; R003 now listed against
  RSK004 (was missing).
- Per-doc dependency graph: connected DAG, no cycles, R001 +
  R002 as fan-in hubs. R004's drop of R005 removes the only
  backward-reading edge S017 surfaced. Improved structure over
  the pre-judge state.
- Statement lines match outcome README verbatim: ✓ (all 6).
- All ACs solution-free (no UI nouns, no thresholds, no scale
  specifications): ✓ — including the new R004 AC #4.
- All statements follow rubric pattern `[Product/Solution] must
  [Capability/Constraint]`: ✓.
- **RSK002 (Stale Status Assessment) — two mitigators (R002,
  R005).** Not a fragility — R002 surfaces the gap, R005 closes
  it. Honest separation of read and write.
- **RSK001 (Bad Status Inference) — three mitigators (R001,
  R003, R004).** Heavy fan-in mirrors O001-R001 hub pattern.
  Standing.

## Vertical & horizontal trace

- **Vertical:** every requirement traces to a mapped risk;
  every risk impact clause closes on the outcome statement
  ("tracked status reflects its true status").
- **Sibling (intra-outcome):** R001 is the basis hub; R002 is
  the freshness hub; both stand alone with no intra-outcome
  deps. R003 → R001; R004 → R001 (after R005 drop); R005 →
  R002; R006 → R001, R003. R004's broadened scope (absorbing
  override-vs-source) does not introduce new intra-outcome
  deps — R004 already depended on R001.
- **Cousin (cross-outcome) — sharpened this session:**
  - **O003-R002 vs O001-R002.** Same shape (per-item
    product-side assessment timestamp), different objects (item
    status vs slip status). Boundary unchanged; PDR pressure
    unchanged.
  - **O003-R005 consumes O002-R006.** Real consumption dep,
    unchanged.
  - **O003-R004 ↔ O002-R002 (Manual Item Capture).** Manual
    items can only carry overridden status. Named in R004's
    edge case 3, not in deps. Unchanged.
  - **O003-R001 ↔ O001-R006 (Source Availability).** Named in
    R001 edge case 3 ("source expected but unread"). Unchanged.
  - **O003-R006 feeds O001-R001.** Source disagreement on
    slip-relevant signals lands in O001-R001's "cannot assess"
    state. Not in deps (R006 doesn't know what's slip-relevant);
    standing flag from S017.

## Deferred (so we don't lose it)

New this session:

- **R001 / R002 / R004 history capability** — basis recoverable
  after override clear (R001 EC1, R004 EC2) plus override
  timestamp distinct from data-update timestamp (R002 EC3)
  jointly assert audit/event-store capability with no owning
  requirement. Same shape as S014's R001/R003/R004 history
  flag — second occurrence across outcomes. Engineering-layer.
- **R002 multi-source picking rule** — now flagged in-doc;
  engineering-layer commitment carried forward.
- **R006 dep on R003 reads as downstream-effect, not
  consumption** — loose dep convention noted; reviewer-fruitful,
  not judge-fruitful.
- **R004 broadened scope (override-vs-source absorption)** —
  reviewer should probe whether the new AC #4 wording ("current
  basis") needs a defined term, or whether "inferred reading
  from current basis" is crisp enough.

Promoted / sharpened PDR-candidates (deferred since S014/S015/
S017; pressure now four sessions deep):

- **"Cannot assess" / coverage-over-precision posture.**
  Unchanged from S017 — still 8+ docs across 3 outcomes; R006's
  "doesn't silently resolve" reinforces the posture without
  adding a new doc. PDR needed before O004/O005 to stop the
  accumulation.
- **Per-item freshness presentation.** Unchanged — 3 docs
  (O001-R002, O002-R006 refresh variant, O003-R002).
- **R007 Source Registration** — unchanged from S015. Two
  literally-duplicated docs (O001-R007, O002-R007).

Carried forward (unchanged):

- **O001-RSK002 single-mitigation fragility** (R001 only).
  Standing from S006.
- **No-owner-at-all seam between O002 and O005.** Standing from
  S012.
- **Override is a four-requirement shape** (R001, R002, R004,
  R005, R006). Standing from S017; reinforced this session —
  R004 now explicitly absorbs override-vs-source surfacing,
  making the four-requirement shape even more component-shaped.
  Likely a component boundary in engineering.
- **Engineering concerns** (unchanged shape; sharpened by R004
  absorption):
  - **Decisioning component** — R005 detection mechanism, R006
    source-vs-source surfacing logic, plus R004's broader
    "current basis vs override" surfacing logic.
  - **Per-item cache with freshness** — R002 two-timestamp
    presentation + R005 reassessment loop + R002's new
    multi-source picking rule.
  - **History / event store** — R001 basis history (override
    recoverable after clear), R004 cleared-override history,
    R002 override-timestamp distinct from data-update timestamp.
- **Missing session 016 (O002 judge)** — accepted as standing
  gap per S017 user decision. Do not re-raise. Method for
  reading 016's findings unchanged: diff S015's described draft
  state against the current O002 R-docs.
- **Workspace path mismatch in Cursor.** Unchanged from S015.

Cleared:

- (none)

## Recommended next step

1. **Commit + PR the four edited files on branch
   `product-docs`** so an independent reviewer can run session
   019 (review) against the Requirement rubric. Reviewer-
   fruitful probes:
   - R004 ↔ R006 scope split — does override-vs-source land on
     R004 cleanly, or should R006 widen and accept the dep on
     R004? (Both options surfaced explicitly to the user this
     session; reviewer may push for the alternative.)
   - R004 new AC #4 wording — is "inferred reading from current
     basis" crisp enough, or does it need a defined term?
   - R002 multi-source clause — engineering-layer punt
     acceptable, or push for an explicit picking rule at the
     requirement layer?
   - R003 → RSK004 mapping addition — accept as map-correctness,
     or push for R003 to acknowledge the role in its own
     dependency list?
   - R003 confidence scale (still unspecified) — held; reviewer
     may push.
   - R005 timing bound to checkpoint cadence — held; reviewer
     may push for clock-interval framing.
   - R005 detection mechanism deferred — held; reviewer may
     push.
2. **PDR session** — the three PDR-candidates ("cannot assess,"
   per-item freshness, source registration) are now four
   sessions deep. S014 / S015 / S017 / S018 all recommended it.
   Doing it before drafting O004/O005 still saves rework — the
   per-requirement docs will be substantially smaller (no
   repeated posture, no repeated freshness shape, no repeated
   registration paragraph).
3. **Then decide on O004 / O005 drafting.** If the PDRs land,
   per-requirement docs will be smaller. If they don't land
   first, the repetition pressure will be at four outcomes deep
   by the end of O004 — retroactive PDR application becomes a
   large rework pass at that point.

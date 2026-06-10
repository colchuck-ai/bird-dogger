# Session 024 — Drafted product PDR004 (Override Surfacing)

## Goal

Open the PDR-drafting session for the override-surfacing pattern
that S023 confirmed ripe. Three confirmed instances across three
outcomes (O003-R004 status override, O004-R003 intervention
recommendation override, O005-R004 escalation timing recommendation
override) — same instance shape that drove PRD001 (3+ outcomes)
and PRD002 (3 R-docs). User came in directly with "let's draft it
now"; no sequencing call to make this session.

## Context loaded

- `assets/templates/pdr-simple.md` from the `intent` skill — same
  template the three existing PDRs used.
- All three existing PDRs:
  `docs/product/pdrs/PRD001-coverage-over-precision.md`,
  `PRD002-per-item-freshness-presentation.md`,
  `PRD003-shared-source-registration.md` — for voice, structure,
  length, citation style, and cross-PDR reference precedent.
- The three R-docs that exhibit the override shape:
  `J001-O003-R004-status-override.md`,
  `J001-O004-R003-intervention-recommendation.md`,
  `J001-O005-R004-escalation-timing-recommendation.md` — for the
  exact wording each carries on override-survives-resynthesis,
  closed-state vocabularies, and the cross-R-doc references
  ("same shape O003-R004 commits to," etc.).
- `docs/product/README.md` — product root for vertical trace and
  outcome list.
- `intent` skill `SKILL.md` (PDR rubric in Records section).
- `resources/previous-sessions/023-requirements-o005-judged.md` —
  the immediate trigger; recommended-next-step #2 is exactly this
  session. Standing PDR-candidate framing inherited from there.
- `resources/previous-sessions/019-product-pdrs-drafted.md` — the
  direct precedent for PDR drafting shape (simple template,
  Context-by-citation, Options balance, no-judge user decision,
  reviewer-only quality gate).

Skipped: did not load O001/O002 R-docs (none exhibit the override
shape; the three relevant R-docs are all in O003/O004/O005 and
were loaded). Did not load engineering or component docs (none
exist). Did not load older session digests beyond S019/S023 —
those two carry the chain forward sufficiently. Did not load
S017/S018 directly even though S023 references them as PDR-pressure
sessions; S023's digest of those flags was sufficient.

## Workspace path mismatch — still standing, tenth session

Tenth session running with the same workaround. Cursor opens
`/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`; real repo
is `/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. First
Shell call (`ls` against the workspace path) failed; `pwd`
confirmed the mismatch. Switched to absolute paths through the
real repo for all reads + writes — same workaround S015 onward.

**Standing for next session:** unchanged — repoint the Cursor
workspace at
`/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. Ten
sessions deep on the workaround now.

## Template choice

Simple template. Same precedent as PRD001/PRD002/PRD003 — no
attached material (no nested CRs, no nested ADRs, no follow-up
artifacts).

## PDR framing decisions

Cross-cutting moves applied:

- **One-sentence summary at the top.** Captures both senses the
  PDR holds: persistent override + disagreement surfaced on
  re-synthesis. Same shape as the three prior PDRs.
- **Context cites concrete R-docs by ID.** Three named with full
  scope of the override shape they each carry, plus inline
  reference to O003-R001 (basis trace) and O003-R002 (timestamp)
  inside the O003-R004 bullet — those are intra-O003
  ratifications of the override, not new instances of the shape,
  so they ride along rather than getting their own bullet.
- **Two coupled commitments named explicitly.** S023's "closed-
  state-enum posture" + "override survives re-synthesis" come
  through as a numbered pair so a reviewer or downstream R-doc
  author sees the shape has two parts that compose.
- **Options name real alternatives that are under pressure.**
  Same balance precedent as PRD001/2/3. Option B
  (override-expires-on-re-synthesis) is the actual default a
  future judge could pick if the posture weren't recorded;
  Option C (per-outcome divergence) is the explicit composition
  failure case — uses PRD001's mixed-posture composition argument
  by analogy.
- **Cross-PDR reference to PRD001 — load-bearing.** PRD004 cites
  PRD001 in two places: once in Option C (composition failure
  argument) and once in Consequences (the two PDRs together
  cover product uncertainty + bird-dogger authority). Same
  forward-cite pattern PRD002 used; same back-reference asymmetry
  flagged for reviewer.
- **Consequences read through to specific downstream concerns.**
  Citation pattern for future R-docs (matches PRD001), closed-
  state-vocabulary obligation on future producers, readability
  obligation on R-docs, cleared-override-recoverable AC pattern,
  composition with PRD001, engineering-layer one-mechanism
  punt to ADR territory, judge/reviewer probe inversion.

PDR-specific notes:

- **Title.** "Override Surfacing" — matches S022/S023 standing
  language. Captures both the override-as-first-class-state and
  disagreement-surfacing-on-re-synthesis senses. Reviewer flag
  noted: "Override Persistence" would emphasize the survives-
  resynthesis piece more sharply; the title leans on the
  rendering side.
- **Closed-state vocabulary, not closed-state enum.** O003-R004
  draws status vocabulary from the basis (R001), not an explicit
  three-state enum. O004-R003 ("intervene now" / "not now" /
  "cannot recommend") and O005-R004 ("escalate now" / "not yet"
  / "cannot recommend") carry explicit three-state enums. The
  PDR uses "closed-state vocabulary" as the umbrella so all
  three fit without mis-claiming a uniform shape. Reviewer flag
  noted: tighter framing or sharper distinction may be pushed.
- **O004-R004 (Triage State Carry-Forward) explicitly out of
  scope.** R003's Detail already names the distinction —
  intervention recommendation produces a closed enum the override
  can target, while triage state carries an open vocabulary the
  bird-dogger supplies. PRD004's Consequences pre-empts the
  reviewer probe: free-text / open-vocabulary judgments are not
  under PRD004's scope and need an explicit deviation. R004's
  posture (open-vocabulary, workflow-owned) stays where it is.
- **Engineering layer punt to ADR.** "One override mechanism
  parameterized by judgment type" — same shape as PRD002's
  picking-rule punt and PRD003's component-consolidation punt.
  Sets the requirement-layer expectation; component layout is
  ADR territory.

## Open: judge / no-judge call on PRD004

S019 explicitly recorded "User decision: no judge session on the
three PDRs" with the consequence chain: reviewer becomes the
only quality gate, boundary-call flags become reviewer flags, no
retroactive judge recovery proposed in future sessions, S023
clears it as "PDR judge pass on PRD001/PRD002/PRD003 — still
explicitly off the table per S019 user decision."

That posture has not been re-asked for PRD004. The user did not
declare it this session, and the assistant did not surface the
question with `AskQuestion`. Two reasonable defaults exist:

- **Inherit the S019 posture.** PRD004 is a sibling PDR drafted
  in the same shape, same template, same drafting-only cycle.
  No-judge applies by default; reviewer is the only quality
  gate.
- **Re-ask.** PRD004 is the first PDR drafted after the four
  Requirement-rubric judge sessions (S014, S018, S021, S023)
  reinforced the value of pre-review rubric checks. The user
  may want a judge pass on PRD004 specifically — and on the
  PDR rubric generally, which has never been judge-tested.

Standing as a question for the next session if it carries forward
to commit/PR. The assistant did not pre-check PRD004 against the
PDR rubric in `intent/SKILL.md` — same posture as S019 carried.

## Boundary calls worth flagging for the reviewer

(Same framing as S019: not held-back judge work; flagged because
the no-judge posture from S019 is the assumed default for PRD004
unless the user reverses.)

- **PRD004 title scope.** "Override Surfacing" emphasizes the
  rendering/disagreement piece. "Override Persistence" would
  emphasize the survives-resynthesis piece. Body covers both;
  the title leans on one. Reviewer may push for a different
  framing.
- **O003-R005 / O003-R001 not listed as instances.** They
  ratify the O003-R004 override (R005 commits to not silently
  replacing; R001 traces it in basis) but were treated as
  intra-O003 ratifications inside the O003-R004 bullet rather
  than separate instances. Reviewer may push for explicit
  listing.
- **Closed-state vocabulary phrasing.** Used as the umbrella
  for O003-R004 (basis-vocabulary) and O004-R003 / O005-R004
  (explicit three-state enums). Reviewer may push for tighter
  language or a sharper distinction between the two cases.
- **PRD004 ↔ PRD001 reference direction.** PRD004 cites PRD001
  (forward-only). PRD001 does not cite PRD004 back. Same
  asymmetry PRD002 carried; same reviewer flag applies.
  Justification: PRD001 is the broader posture; PRD004 is one
  layer of how authority composes with uncertainty. Reviewer
  may push for back-references on PRD001 (and PRD002) for
  symmetry.
- **O004-R004 named as out-of-scope.** Pre-empts a probe but
  commits PRD004 to a particular reading of R004's scope (open-
  vocabulary, workflow-owned, not under the override shape).
  Reviewer may push back if O004-R004's open-vocabulary posture
  is itself under-specified or contested.
- **Options balance.** Same flag S019 raised on PRD001/2/3:
  the PDR template doesn't require options to be balanced
  (some real, some strawmen). Option C (per-outcome divergence)
  leans on PRD001's composition argument by analogy; reviewer
  may push for sharper steelmanning.

## What got written

One file under `docs/product/pdrs/`:

- `PRD004-override-surfacing.md` — 149 lines, simple template,
  Context (with three concrete R-doc citations and the
  two-commitments breakout), Options (three alternatives),
  Decision (rationale grounded in J001 outcome composition),
  Consequences (seven bullets: citation pattern, closed-state-
  vocabulary obligation, readability obligation, cleared-
  override-recoverable AC pattern, composition with PRD001,
  engineering-layer punt, judge/reviewer probe inversion).

PDR is longer than PRD001 (103), PRD002 (89), PRD003 (84) —
justified by dual scope (behavior commitment + closed-state
vocabulary constraint) plus explicit composition with PRD001.
Length flagged in chat; user can trim during review.

No Change Record — drafting in a create cycle (S005 / S007 /
S009 / S011 / S013 / S015 / S017 / S019 precedent extended to
this fourth PDR).

## Coverage check

- PDR-candidate from S023 is now drafted: ✓ (override surfacing,
  promoted from "ripe" to "drafted" this session).
- PDR cites at least one R-doc by ID in its Context: ✓ (three
  R-docs primary, two R-docs intra-O003 ratification).
- Decision is unambiguous (single named option, not hedged): ✓
  (Option A — persistent override with surfaced disagreement).
- Consequences include both positive and negative or follow-up:
  ✓ (citation pattern enables; closed-state-vocabulary obligation
  constrains; readability obligation costs surface area;
  engineering-layer punt is follow-up).
- No PDR contradicts existing R-doc text. PRD004 codifies what
  is already there in O003-R004, O004-R003, O005-R004; does not
  introduce new commitments at the requirement layer.
- File-naming convention matches the SKILL.md `Files` section:
  product-level path `docs/product/pdrs/PRD<NNN>-<name>.md`: ✓.
- PDR numbering continues from PRD003: ✓ (PRD004).
- Cross-PDR reference uses relative-link form
  `[PRD001](PRD001-coverage-over-precision.md)`: ✓ (matches
  PRD002's pattern).

## Vertical & horizontal trace

- **Vertical:** PRD004 traces to three outcomes via three R-docs
  (O003-R004, O004-R003, O005-R004). The override shape composes
  across outcomes per the Decision (status feeds urgency feeds
  escalation timing; an override at one layer that gets silently
  replaced breaks the layers above it).
- **Horizontal (between PDRs):**
  - **PRD004 ↔ PRD001** — load-bearing forward reference from
    PRD004 to PRD001, in two places: Option C (composition
    failure argument by analogy) and Consequences (the two
    postures together cover product uncertainty + bird-dogger
    authority). Reviewer flag on direction noted above.
  - **PRD004 ↔ PRD002** — no edge. PRD002 is freshness
    presentation; PRD004 is judgment override. Independent
    decisions on independent topics. The recommendation R-docs
    O004-R003 and O005-R004 consume freshness inputs (PRD002
    territory) and produce overridable judgments (PRD004
    territory) — but the consumption / production layers don't
    require cross-PDR reference at the PDR layer.
  - **PRD004 ↔ PRD003** — no edge. PRD003 is shared-capability
    requirement duplication; PRD004 is product-produced
    judgment override. No load-bearing connection.

## Deferred (so we don't lose it)

New this session:

- **Judge / no-judge call on PRD004 — open.** S019 set
  no-judge-on-PDRs; S023 cleared the PDR1/2/3 judge pass as
  off the table per that decision. PRD004 has not had the
  question re-asked. Defaults discussed in the open call
  section above. Next session call.
- **PRD004 ↔ PRD001 back-reference.** PRD001 currently cites
  no other PDRs. Adding a back-reference to PRD004 in PRD001's
  Consequences (and to PRD002 for symmetry) would close the
  reference asymmetry flagged in S019 and again here. Update-
  cycle work with a CR scoped to PRD001 if pursued.
- **O004-R004 open-vocabulary posture.** PRD004 explicitly
  carves R004 out as open-vocabulary, workflow-owned. R004's
  R-doc itself does not currently declare that posture in its
  own Detail (the open-vocabulary framing lives only in
  O004-R003's contrast paragraph). Reviewer may push for a
  small sharpening edit on R004 to make the posture local-
  readable. Update-cycle work if pursued.

Promoted / sharpened PDR-candidates:

- **Override surfacing** — promoted from S023's "drafting
  candidate" to drafted. Closed.

Carried forward (unchanged or sharpened):

- **PRD001 / PRD002 / PRD003 cited rather than re-derived.**
  PRD004 leans on PRD001's composition argument by analogy
  (Option C); cites PRD001 directly in two places. Working as
  intended.
- **PRD003 (shared source registration)** — still uncited
  under O005. R006 boundary call (escalation contact
  registration) still open.
- **R005 dual role on one store** (S023). Engineering-layer
  ADR territory when the engineering layer opens.
- **Risk map vs Detail-level mitigation roles** (S023). Same
  posture call shape; convention emerging.
- **"Cousin-of-shape, not consumption" as a citation pattern**
  (S022/S023). Documentation convention; not PDR-shaped.
- **Engineering concerns sharpened by this session:**
  - **Override mechanism** — PRD004 commits to one mechanism
    parameterized by judgment type, not three. Sharpest
    component-boundary flag for the engineering layer yet on
    overrides. ADR territory.
  - **Decisioning component** — unchanged from S023 (synthesis
    hub for O004 and O005).
  - **History / event store** — unchanged from S023 (R005
    dual-role escalation + touch).
  - **Identity / chain store, per-item cache, dependency graph
    store, source registration component** — unchanged.
- **Override is a multi-requirement shape within O003** (S017/
  S018 standing: O003 R001/R002/R004/R005/R006 all touch the
  status override). PRD004 captures the cross-outcome shape;
  the intra-O003 web is referenced inline under the O003-R004
  bullet. Engineering-layer flag still standing.
- **O001-RSK002 single-mitigation fragility.** Standing from
  S006. Not touched.
- **No-owner-at-all seam between O002 and O005.** Standing
  from S012. Not touched.
- **Missing session 016 (O002 judge).** Standing user
  decision: do not recover, do not re-run. Unchanged.
- **Workspace path mismatch in Cursor.** Ten sessions running
  with the absolute-path workaround.

Cleared this session:

- **PDR4 (override surfacing) drafting candidate** — drafted.
  Closed.

## Recommended next step

1. **Decide judge / no-judge on PRD004.** Two paths:
   - **Inherit S019 no-judge posture.** Treat PRD004 the same
     as PRD001/2/3; reviewer is the only quality gate;
     boundary calls in this digest become reviewer flags.
     Lowest overhead, consistent with prior PDR drafts.
   - **Run a judge pass on PRD004 (and/or the PDR rubric
     generally).** First PDR drafted after four
     Requirement-rubric judge sessions reinforced the value
     of pre-review rubric checks. The PDR rubric in
     `intent/SKILL.md` has never been judge-tested. Higher
     overhead; surfaces rubric-correctness defects before
     reviewer.
   AskQuestion-shaped call; defer to user.

2. **Commit + PR PRD004 on branch `product-docs`** so a
   reviewer can run a review session against the PDR rubric.
   Bundle options:
   - **Bundle with the still-deferred O005 review** (S023
     recommended-next-step #1). Same branch, related scope —
     PRD004 codifies the override shape O005-R004 carries.
     Reviewer can probe both together.
   - **Bundle with O004 + O005 review** if S021's still-
     deferred O004 review also rolls up. Three artifacts in
     one review session.
   - **Or split** — PRD004 review separate from R-doc reviews.
     Costs more session overhead; benefits clarity.
   Reviewer-fruitful probes for PRD004 listed in boundary
   calls section above.

3. **Post-PR follow-ups (update cycles if accepted):**
   - PRD001 + PRD002 back-reference to PRD004 for symmetry —
     CR-scoped update cycle.
   - O004-R004 sharpening to declare the open-vocabulary
     posture locally — CR-scoped update cycle.

4. **PDR judge pass on PRD001/PRD002/PRD003** — still
   explicitly off the table per S019 user decision (cleared
   in S023). Do not propose. Reviewer feedback on those PDRs
   may merit changes; those go through update cycles with
   CRs scoped to the changed PDR(s).

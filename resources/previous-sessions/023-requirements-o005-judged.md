# Session 023 — Judged requirement documents for O005 Escalation Quality

## Goal

Run the judge pass on the five per-requirement documents under
`docs/product/outcomes/J001-O005-escalation-quality/requirements/`
(drafted in S022) against the Requirement rubric in the `intent`
skill, before going to review. Fourth judge session below the
outcome layer after S014 (O001), S018 (O003), and S021 (O004).
Second R-doc judge session after the three product PDRs landed;
S021 set the precedent for resolving S020-style standing prompts
inline + applying soft edits inside the judge pass. Session ran
in two halves: judge pass first, then user said "apply
recommendations from judge" and edits were applied in the same
session — same shape as S021.

## Context loaded

- All 5 R-docs under
  `docs/product/outcomes/J001-O005-escalation-quality/requirements/`
  — the drafts under judgment.
- `docs/product/outcomes/J001-O005-escalation-quality/README.md`
  — owning outcome; risk-requirement map for coverage check.
- `docs/product/README.md` — product root for vertical trace.
- Two product PDRs:
  `docs/product/pdrs/PRD001-coverage-over-precision.md`,
  `PRD002-per-item-freshness-presentation.md` — fully loaded for
  citation-correctness check. PRD003 not loaded; no O005 R-doc
  cites it (consistent with S022 noting no shared-capability
  requirement under O005 as drafted).
- Cousin requirement docs for declared and implied cross-outcome
  edges:
  - `J001-O003-R002` (status freshness) — R002 cousin-of-shape.
  - `J001-O003-R004` (status override) — R004 override-shape
    cousin.
  - `J001-O004-R003` (intervention recommendation) — R004
    declared dep; the "intervention window" naming drift surfaced
    against this doc.
  - `J001-O004-R004` (triage state carry-forward) — R005
    cousin-of-shape (memory hub, event-log divergence).
  - `J001-O001-R001` (proactive slip surfacing) — R004 declared
    dep.
- `intent` skill: `SKILL.md` (Requirement rubric, plain language +
  candor guidance), `references/context-tracing.md` (Requirement
  self/siblings/cousins), `references/workflows.md` (judge inside
  create cycle has no CR), `assets/templates/requirement-simple.md`.
- `resources/previous-sessions/022-requirements-o005-drafted.md`
  — drafting session; the seven standing judge prompts. S021
  consulted as the direct judge-precedent (same session shape:
  judge + apply in one session).
- `git log` confirming commit `bd528aa docs: draft j1 05 reqs`
  on branch `product-docs` between S022 and this session: the
  five new R-docs were committed before the judge ran. Working
  tree was clean before this session's edits.

Skipped: Records (CRs, ADRs) per workflows guidance — judge on
a create cycle. Did not re-load O001/O002 cousin docs the judge
didn't materially need; relied on outcome README + S022/S021
digests for cousin context where appropriate.

## Workspace path mismatch — still standing, worked around

Ninth session running with the same workaround. Cursor opens
`/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`; real repo
is `/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. First
Shell call (`ls` against the workspace path) failed; `pwd`
confirmed the mismatch. Switched to absolute paths through the
real repo for all reads + writes — same workaround
S015/S017/S018/S019/S020/S021/S022 used. `git status` on
`bird-dogger/` on branch `product-docs` (tracking
`origin/product-docs`) shows the three modified files plus this
digest as the only changes from this session.

**Standing for next session:** unchanged — repoint the Cursor
workspace at
`/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. Nine
sessions deep on the workaround now.

## Verdict

Three R-docs (R001, R002, R003) passed clean. Two had defects:
R004 carried a naming drift (statement + Detail both said
"intervention window," but O004-R003 produces a recommendation
state, not a window) and a mechanical dependency-list gap
(R002 named as a synthesis input in Edge Cases but omitted from
Dependencies). R005 had a coupled coherence defect: the
one-line statement was scoped to escalation attempts while
Detail extended R005 to also store routine touches that anchor
R003's response window — statement narrower than actual scope,
and Acceptance Criteria under-tested the touch case.

Five soft edits across three files, all sharpening passes. No
new fragility surfaced beyond what S022 had already flagged.
Shape closest to S021 — dependency-declaration gap (R004) +
statement/scope coherence gap (R005). No AskQuestion: every
defect had a clear surgical fix (rename, add dep, broaden
statement, add AC, attribute risk roles in Detail) and the
S022/S021 precedent on dual-role recording covered the R005
boundary call.

## Standing judge prompts from S022 — resolutions

1. **Override surfacing PDR pressure (promoted) — hold this
   session; PDR drafting candidate is now ripe.** S021 held the
   pattern at two instances (O003-R004, O004-R003). S022's
   drafting of O005-R004 added the third (escalation timing
   recommendation override). Three instances across three
   outcomes is the same shape that drove PRD001 (3+ outcomes)
   and PRD002 (3 R-docs). Judge did not propose PRD004 inline
   — this judge's scope is the five O005 R-docs' rubric
   coherence, not new PDR drafting. The PDR-drafting decision
   is a separate session call; flagged in deferred. No edit.

2. **R005 event-log shape vs O004-R004 single-decision shape —
   accept divergence.** The README's R005 wording ("record each
   escalation attempt") is plural-event; R005's Detail already
   names the divergence explicitly. Judge's R005 critique was
   scope-coherence (touch + attempt merge), not the event-log
   shape. The divergence stands. No edit on event-log framing.

3. **R004 escalation downstream of intervention via O004-R003 —
   accept tier dependency.** Judge accepted the tier framing
   ("escalation is the next-tier action when intervention is
   already established as warranted") in R004's Dependencies
   list. The only R004 fix on the O004-R003 edge was the
   naming drift (see N1). Framing held.

4. **R006 escalation contact registration — hold; not added.**
   Judge did not propose R006 as a new requirement. Scope of
   the judge is rubric coherence of the five existing
   requirements; whether the R-list should grow to 6 is a
   create-cycle decision, not a judge call. PRD003
   Consequences flagged R006 as "a plausible analogue" — that
   probe still sits open for reviewer or for a future drafting
   session. No edit. Flagged in deferred.

5. **R002 PRD002 non-transfer note — accept inline cite-to-flag.**
   Judge passed R002 clean. The non-transfer note (citing
   PRD002 to flag where its shape does not apply) reads as
   honest framing; cousin-pattern is named without a false
   inheritance claim. Reviewer may still prefer this lives as
   a CR on PRD002; left in-doc per S022 author choice. No
   edit.

6. **R003 + R005 touch/attempt merge at the recording layer —
   accept one store with a label; sharpen R005.** Judge
   accepted the one-store-with-label decision but found R005's
   one-line statement and ACs did not reflect it. Fix landed
   on R005 (statement broadened, Detail risk-role attribution
   added, new AC for routine touches). R003 left unchanged —
   R003's "depends on R005" framing is honest about the
   shared store. See N3 / N4 / N5.

7. **R004 dep count (5 deps including 2 cross-outcome) —
   accept and extend to 6.** R004 Dependencies grew by one
   this session (R002 added per N2). Same precedent as S021
   prompt 3 extending R002 from 3 to 5. Every edge is
   load-bearing; R004 is the synthesis hub for O005 and
   carries the high dep count by design. No further trim.

## New issues the judge surfaced

**N1. R004 naming drift: "intervention window" is not the name of
any upstream element — soft edit applied.** R004's statement
listed three input signals ("slip status, intervention window,
response window"). Detail repeated the phrasing
("the intervention window (O004-R003)"). But O004-R003 is named
**Intervention Recommendation** and produces a closed-set state
("intervene now" / "not now" / "cannot recommend"), not a window.
"Intervention window" was an invented label that would drift
downstream readers off the trace. Edited: renamed to
"intervention recommendation" in the statement and the Detail
paragraph that enumerates synthesis inputs. Outcome README
summary line synced.

**N2. R004 Dependencies omitted R002 despite Detail naming
R002 as an input — soft edit applied.** R004 Edge Case 3 reads
"Owner contact is stale (R002 shows long-unconfirmed owner):
this does not block a recommendation, but feeds the synthesis
as reduced confidence; R004 surfaces 'cannot recommend' if the
staleness leaves no usable target signal." That makes R002 a
load-bearing synthesis input. Dependencies omitted R002. Same
mechanical-gap shape as S021 N1 (R002 missing O001-R001 +
O003-R003 despite Detail naming them). Added R002 to R004
Dependencies with one-line consumption framing
("owner-confirmation staleness feeds synthesis as reduced
target confidence; R004 surfaces 'cannot recommend' when
staleness leaves no usable target signal").

**N3. R005 one-line statement narrower than R005's actual scope
— soft edit applied.** Statement said "The product must record
each escalation attempt — target, channel, time, and response —
and surface that history at the item level." Detail extended
R005 to also record routine non-escalation touches ("the
distinction between 'escalation attempt' and 'touch' is a label
on the record, not a separate store") because R003 reads
last-touch from R005's store. A reader of the one-line would
expect R005 to only record escalation attempts and be surprised
that R003 depends on it for routine touches. Edited: broadened
the statement to "each bird-dogger touch on an item — including
escalation attempts — with target, channel, time, and response."
Outcome README summary line synced. Same "scope-statement
alignment" precedent as S018's R005 single-event-vs-cycle
sharpening.

**N4. R005 risk-role attribution implicit in Detail — soft edit
applied.** R005 maps to RSK004 in the README. But Detail now
extends R005 to also serve R003's RSK002 mitigation (via the
touch record). The rubric's "exists only to mitigate the named
risks" check needed the dual role to be named visibly so a
reviewer can verify the scope without inferring it. Edited:
added a sentence in Detail naming the two roles — "the
escalation-attempt recording is R005's direct mitigation of
RSK004, and the routine-touch recording is enabling
infrastructure for R003's mitigation of RSK002. Both roles
share the same record shape; the label is what tells them
apart." Risk-requirement map in the README unchanged — R005
is still RSK004's mitigator; the touch role is enabling-for-R003,
not a new direct risk mitigation.

**N5. R005 Acceptance Criteria under-tested the touch case —
soft edit applied.** Three existing ACs read "escalation
attempt." If routine touches are in R005's scope (per Detail),
ACs needed to cover them too — otherwise R005 could pass its
ACs while quietly failing R003's dependency on it. Edited:
added a new AC ("Each routine touch is recorded with the same
shape and is distinguishable from an escalation attempt by a
label on the record, so R003 can read it as a touch.") and
broadened the visibility AC from "escalation history" to
"touch and escalation history."

## What got edited

Five edits, three files. Bullet form:

- `J001-O005-R004-escalation-timing-recommendation.md`
  - Statement (¶1): renamed "intervention window" to
    "intervention recommendation." Closes N1.
  - Detail (¶2): same rename inside the synthesis-input
    enumeration. Closes N1.
  - Dependencies: added `J001-O005-R002` with consumption
    framing. Closes N2.

- `J001-O005-R005-escalation-record.md`
  - Statement (¶1): broadened from "each escalation attempt" to
    "each bird-dogger touch on an item — including escalation
    attempts." Closes N3.
  - Detail (¶2): added explicit dual-mitigation sentence
    (escalation-attempt → RSK004 direct; routine-touch →
    enabling for R003's RSK002 mitigation; one store, two
    roles, label distinguishes). Closes N4.
  - Acceptance Criteria: added new AC for routine touches;
    broadened visibility AC to "touch and escalation history."
    Closes N5.

- `J001-O005-escalation-quality/README.md`
  - R004 summary bullet: "intervention window" →
    "intervention recommendation."
  - R005 summary bullet: synced to the broadened one-line.

**Not edited:**
- `J001-O005-R001-escalation-target.md` — passed clean.
- `J001-O005-R002-owner-currency.md` — passed clean.
- `J001-O005-R003-response-window-visibility.md` — passed clean.
  Borderline note: Detail says R003 reads last-touch from R005's
  store, which is mechanism-adjacent for a requirement, but
  defensible because R005 is a requirement layer pointer, not
  an implementation layer one. Held.
- Risk-requirement map in the outcome README — unchanged.
  R005's dual role is named in R005's Detail, not in the map
  (R005 still mitigates RSK004 directly; the touch role is
  enabling, not a direct risk-mitigation claim).

No Change Record — judge inside a create cycle (S006 / S008 /
S010 / S012 / S014 / S018 / S021 precedent, `workflows.md`).

## Coverage check

- Risk-requirement map intact: ✓ all 4 risks covered; all 5
  requirements mapped; map unchanged from S022. RSK001 → R001,
  R002; RSK002 → R003, R004; RSK003 → R004; RSK004 → R005.
- Per-doc dependency graph: still a connected DAG, no cycles.
  R001 and R005 remain intra-outcome hubs (zero intra-outcome
  deps). R002 → R001. R003 → R005. R004 → R001, R002 (new),
  R003, R005 (six total including cross-outcome). Cross-outcome
  edges unchanged: R004 → O004-R003 (intervention
  recommendation), R004 → O001-R001 (slip surfacing).
- Statement lines match outcome README verbatim: ✓ (R004 and
  R005 statements + README summaries co-edited this session
  so they stay in lockstep).
- All ACs solution-free: ✓ — new R005 AC names "label on the
  record" (shape) rather than UI mechanism (color, badge, tab).
  Visibility AC broadened in scope only; no new mechanism named.
- All statements follow rubric pattern `[Product/Solution] must
  [Capability/Constraint]`: ✓.
- PDR citations consistent with PDR Decisions: ✓ unchanged.
  PRD001 cited in R001/R002/R003/R004/R005 Edge Cases; PRD002
  cited in R002 (non-transfer flag), R003 (freshness-cousin
  framing). PRD003 still not cited (no shared-capability
  requirement under O005 as drafted; R006 boundary still open).
- **R005 dual role — risk-coverage hygiene.** The risk map
  still names R005 only against RSK004; R005's enabling role
  for R003 is documented in R005 Detail but not claimed as
  a direct mitigation. "Exists only to mitigate the named
  risks" check holds: R005's direct mitigation is RSK004; the
  touch-recording is infrastructure that R003 consumes for its
  own RSK002 mitigation.

## Vertical & horizontal trace

- **Vertical:** every requirement traces to a mapped risk;
  every risk impact clause closes on the outcome statement
  ("escalations land with the right person at the right
  time"). Unchanged.
- **Sibling (intra-outcome) — sharpened this session:**
  - R004 ↔ R002 now declared on R004's side (was implicit in
    Detail, now explicit in Dependencies).
  - R005 dual role (RSK004 direct + R003-enabling for RSK002)
    now named in Detail and reflected in ACs. The touch-vs-
    attempt distinction stays a label on the record per
    S022's framing.
  - R001 / R002 / R003 unchanged.
- **Cousin (cross-outcome) — touched this session:**
  - **R004 → O004-R003 (intervention recommendation)** — name
    now matches the upstream element (was "intervention
    window"). Same consumption-tier framing as S022.
  - **R005 cousin-of-shape to O004-R004 (memory hub)** —
    unchanged from S022. Event-log vs single-decision
    divergence still held; S022 standing prompt 2 resolved
    as accept.
  - **R004 override-shape cousin to O003-R004 + O004-R003** —
    unchanged. Third instance of override-survives-resynthesis
    confirmed by this judge; PDR pressure now ripe (see
    deferred).
  - **No-owner-at-all seam between O002 and O005** —
    unchanged from S022. R001 EC1 still sits on the O005 side
    of the seam.

## Deferred (so we don't lose it)

New this session:

- **Override surfacing — PDR-drafting candidate now ripe.**
  Three confirmed instances across three outcomes (O003-R004
  status override, O004-R003 intervention recommendation
  override, O005-R004 escalation timing recommendation
  override). Same instance-count shape that drove PRD001 and
  PRD002. This judge did not action a PDR draft (out of scope
  for a rubric judge); next drafting session is a candidate
  for PRD004 if the user wants to compress the pattern. If
  drafted, downstream R-docs can cite rather than re-derive
  the override-survives-resynthesis shape.

- **R005 dual role on one store — engineering layer
  implication sharpened.** R005 now explicitly carries two
  mitigation roles. The engineering-layer realization is one
  store with a label field distinguishing touch from attempt.
  ADR territory when engineering opens.

- **Risk map vs Detail-level mitigation roles — convention
  emerging.** R005 is the first requirement in J001 to carry
  a documented dual role (direct mitigation of one risk +
  enabling infrastructure for another requirement's mitigation
  of a different risk). The convention this session adopts:
  the README risk-requirement map names only direct
  mitigations; enabling roles live in the requirement's
  Detail. Reviewer may push to lift this into the map (e.g.,
  marking R005 as RSK002-enabling) or hold as Detail-only.
  Same posture call shape as S021 N5 (RSK001 → R003 mapping
  rationale carried in Detail, not in the README description).

Promoted / sharpened PDR-candidates:

- **Override surfacing** — promoted from S022's "ripe but not
  drafted" to "drafting candidate this session confirmed."
  See above.

- **"Cousin-of-shape, not consumption" as a citation pattern**
  — unchanged from S022. Still a documentation convention,
  not a posture decision; not PDR-shaped.

Carried forward (unchanged or sharpened):

- **PRD001 / PRD002 cited rather than re-derived.** Continued
  working as intended through this judge pass; no R-doc was
  dinged for re-stating posture, and PDR citations didn't
  surface contradictions.

- **PRD003 (shared source registration)** — still uncited
  under O005. R006 boundary call (escalation contact
  registration) still open as a possible future requirement
  per PRD003 Consequences.

- **Engineering concerns sharpened by this session:**
  - **Decisioning component** — R004 confirmed as the
    synthesis hub for O005; six dependency edges feed it.
    Spans four outcomes now (O001/O002/O003/O004 already
    flagged; O005-R004 confirms).
  - **History / event store** — R005 now explicitly carries
    a dual role (escalation attempts + routine touches in
    one store). Sharpest version yet of the audit/event-store
    pressure; clearer ADR shape than S022 surfaced.
  - **Identity / chain store** — unchanged from S022 (R001
    per-item owner + chain-above representation).
  - **Per-item cache with freshness** — unchanged from S022
    (R002 owner-confirmation, R003 last-touch elapsed).
  - **Dependency graph store** — unchanged from S020/S022.

- **R001/R002/R004 history capability** — R005 in this session
  is the most explicit event-log requirement in J001.
  Engineering-layer.

- **Override is a multi-requirement shape** — three confirmed
  instances now; PDR pressure ripe (see promoted above).

- **O001-RSK002 single-mitigation fragility.** Standing from
  S006. Unchanged.

- **No-owner-at-all seam between O002 and O005.** Standing
  from S012. Touched by S022 (R001 EC1 on the O005 side);
  unchanged this session.

- **Missing session 016 (O002 judge).** Standing user
  decision: do not recover, do not re-run. Unchanged.

- **Workspace path mismatch in Cursor.** Nine sessions
  running with the absolute-path workaround.

Cleared this session:

- **All seven S022 standing judge prompts** — resolved above
  (1 → hold; PDR-drafting candidate flagged. 2 → accept.
  3 → accept. 4 → hold. 5 → accept. 6 → accept-and-sharpen
  R005. 7 → accept-and-extend to 6 deps.)

## Recommended next step

1. **Commit + PR the three edited O005 files on branch
   `product-docs`** so an independent reviewer can run a
   review session against the Requirement rubric. Bundle
   options:
   - **Bundle with S021's still-deferred O004 review** —
     S021's recommended-next-step #1 (review of O004 edits)
     never ran as a separate session. O004 was committed
     (`3a09584 docs: judge j1 o4 requirements`) but no
     review session followed. Bundling O004 + O005 review
     into one session is reasonable: same rubric, similar
     scope, and the override-surfacing PDR pressure shows
     up in both outcomes.
   - **Or split** — O004 review separate, O005 review
     separate. Costs more session overhead; benefits clarity
     of which feedback applies to which outcome.

   Reviewer-fruitful probes for O005:
   - R004 "intervention recommendation" rename — accept
     as the load-bearing fix it was meant to be, or push for
     additional sharpening of the synthesis-input
     enumeration?
   - R005 dual-role attribution in Detail vs the risk map —
     accept Detail-only documentation, or push to mark R005
     as RSK002-enabling in the README?
   - R005 new touch AC — does "distinguishable by a label on
     the record" land, or does it need a defined term?
   - R005 statement broadening — accept the
     "touch-including-attempts" framing, or push for separate
     records / separate requirements?
   - R004 six-dep edge count — accept as load-bearing
     synthesis hub per S021 R002 precedent, or push for
     trim?
   - Standing-hold reviewer probes (carried from S022):
     R005 event-log shape (lift or hold), R006 escalation
     contact registration (add or hold), R002 PRD002
     non-transfer note (inline or CR), R004 escalation-
     downstream-of-intervention framing (tier or parallel).

2. **PDR004 (override surfacing) — drafting candidate.** Three
   confirmed instances across three outcomes (O003-R004,
   O004-R003, O005-R004). Same instance shape that drove
   PRD001 and PRD002. The next drafting session could open a
   PRD004 capturing: the closed-state-enum posture, the
   "override survives re-synthesis" shape, and the in-doc
   citation pattern that downstream R-docs use. Sequencing
   choice: draft PRD004 before O005 review (so review can
   probe whether the PDR replaces in-doc duplication), or
   after (so review feedback shapes the PDR's scope). User
   call.

3. **PDR judge pass on PRD001/PRD002/PRD003** — still
   explicitly off the table per S019 user decision. Do not
   propose. Reviewer feedback may merit changes; those go
   through update cycles with CRs scoped to the changed
   PDR(s).

# Session 022 — Drafted requirement documents for O005 Escalation Quality

## Goal

Use the `intent` skill to draft the per-requirement documents for
the next important outcome under J001. S021's recommended-next-
step #2 named O005 explicitly; user's opening ask ("draft
requirements for the next important outcome") matched. Second R-doc
drafting session after the three product PDRs landed; the first
(S020) drafted O004 and set the PDR-citation format precedent that
S021's judge pass validated.

## Order-of-operations note — S021's #1 skipped

S021's recommended-next-step #1 was "commit + PR the five edited
O004 R-doc files on branch `product-docs` so an independent
reviewer can run session 022 (review) against the Requirement
rubric." User moved straight to #2 (this session) instead. The
O004 R-doc edits from S021 are committed (HEAD is
`3a09584 docs: judge j1 o4 requirements` on branch
`product-docs`, tracking `origin/product-docs`, clean before this
session). Review session on O004 is still outstanding and should
run independently of this session's output. Flagged as deferred,
not lost.

## Context loaded

- `docs/product/README.md` — product root; J001 + five outcomes.
- `docs/product/outcomes/J001-O005-escalation-quality/README.md`
  — outcome under drafting; statement lines and risk-requirement
  map untouched, lifted verbatim.
- `docs/product/outcomes/J001-O004-triage-speed/README.md` —
  sibling and primary cousin source (R004 here is shape-cousin
  to O004-R003 per S020/S021 flags).
- All three PDRs:
  `docs/product/pdrs/PRD001-coverage-over-precision.md`,
  `PRD002-per-item-freshness-presentation.md`,
  `PRD003-shared-source-registration.md` — fully loaded so the
  R-docs could cite rather than re-derive. PRD003 Consequences
  flagged "escalation contact registration in O005 is a plausible
  analogue" — held; see boundary call below.
- Sample requirement files for in-doc precedent:
  - `J001-O004-R001-urgency-ranking.md` (hub pattern),
  - `J001-O004-R003-intervention-recommendation.md` (closed-enum
    recommendation + override-survives-resynthesis shape — the
    direct cousin of this session's R004),
  - `J001-O004-R004-triage-state-carry-forward.md` (memory hub
    pattern — the direct cousin of this session's R005),
  - `J001-O003-R002-status-freshness.md` (PRD002 freshness
    pattern — cousin-of-shape for R002 here),
  - `J001-O003-R004-status-override.md` (the canonical override
    shape; load-bearing for R004 here),
  - `J001-O003-R006-conflict-surfacing.md` (multi-source posture
    referenced for R001 chain-disagreement EC),
  - `J001-O004-R002-list-level-urgency-signals.md` (list-level
    framing).
- `intent` skill (attached to the user's prompt): `SKILL.md`
  (Requirement rubric, plain language + candor guidance),
  `assets/templates/requirement-simple.md`.
- `resources/previous-sessions/020-requirements-o004-drafted.md`,
  `021-requirements-o004-judged.md` — drafting + judge precedent.
  PDR-citation format (Edge Cases inline) and override-shape PDR
  pressure carried forward as load-bearing context.

Skipped: Records (CRs, ADRs) per workflows guidance — drafting
in a create cycle. Did not load O001/O002/O003 R-doc files
beyond the two listed; relied on summaries in O004 R-docs and
on the cross-outcome READMEs already loaded.

## Workspace path mismatch — still standing, worked around

Eighth session running with the same workaround. Cursor opens
`/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`; real repo
is `/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. First
Read tool call against the workspace path failed; `pwd` confirmed
the mismatch. Switched to absolute paths through the real repo
for all reads + writes — same workaround
S015/S017/S018/S019/S020/S021 used. `git status` on
`bird-dogger/` on branch `product-docs` shows the five new R-doc
files (untracked, under the new
`docs/product/outcomes/J001-O005-escalation-quality/requirements/`
directory) plus this digest as the only changes from this session.

**Standing for next session:** unchanged — repoint the Cursor
workspace at
`/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. Eight
sessions deep on the workaround now.

## Outcome selection

User asked for "the next important outcome." S021 named O005
explicitly (and S020 before it). Only outcome lacking individual
R-files — confirmed by directory listing before drafting. No
AskQuestion this session: selection was unambiguous and the
PDR-citation format precedent (S020/S021) made the second branch
S020 asked about ("cite + skip vs. cite + restate") moot. Worth
flagging that the no-question call differs from S020's two-
question opener; same precedent as S017's no-question drafting
once the format was clear.

## Template choice

Simple template for all 5. None of the O005 requirements carry a
PDR, CR, or other attached material yet (the three product PDRs
are referenced, not nested under O005). Same precedent as
O001/O002/O003/O004.

## Requirement framing decisions

Statement lines lifted verbatim from the outcome README's
requirements list — no rewording. Same trace-preservation pattern
as S013 / S015 / S017 / S020.

Cross-cutting moves applied across the 5:

- **PDR citations inline in Edge Cases per S020/S021 precedent.**
  PRD001 cited in every R-doc (R001 EC1 "no owner known," EC2
  "no chain," EC3 chain disagreement; R002 EC1 "never confirmed";
  R003 EC1 "no touch"; R004 EC1 "cannot recommend"; R005 EC1
  "delivery failure," EC3 "unrecorded target"). PRD002 cited
  *explicitly to flag non-transfer* in R002 (see candor note
  below) and *to anchor freshness* in R003 cousin framing. PRD003
  not cited — see boundary call.
- **Candor on shape mismatch (R002).** PRD002 commits to a
  two-timestamp shape (assessed + underlying-data-change). For
  owner currency, there is no analogue of "underlying owner-
  change time" most sources expose. Rather than silently adopt a
  one-timestamp variant or assert PRD002 covers this, R002's
  Detail names the non-transfer explicitly and commits to
  confirmation-time only. First instance of an R-doc citing a
  PDR to flag where its shape does not apply.
- **R004 reuses O004-R003's shape language verbatim where
  appropriate** ("survives re-synthesis on later checkpoints in
  the same shape O003-R004 and O004-R003 commit to"). Third
  instance of the override-survives-resynthesis pattern; PDR
  pressure now real — see deferred section.
- **R005 framed as event log, not single-decision state.** The
  README's R005 description ("record each escalation attempt")
  is plural-event by wording. O004-R004 (the memory-hub precedent)
  is single-decision per cycle. R005's Detail names the
  divergence explicitly so a judge or reviewer doesn't read
  R005 as a direct port of O004-R004.
- **Acceptance criteria stay solution-free.** No UI nouns
  (deferred display modes, lists vs. badges), no channel
  taxonomies, no thresholds. S013/S015/S017/S020 precedent.
- **R-docs shorter than O004.** Five R-docs averaged ~70 lines
  each vs O004's ~75 and O003's ~85. PDR-citation savings hold;
  this session also held descriptive paragraphs tighter where the
  cousin precedent already exists (R004 didn't re-explain the
  closed-enum rationale that O004-R003 carries; cited the shape
  instead).

Per-requirement notes:

- **R001 (Escalation Target).** Chain hub for O005. Mitigates
  RSK001. No intra-outcome deps. Three Edge Cases — no owner, no
  chain above owner, chain disagreement across sources — all
  PRD001 surfacings. EC3 (chain disagreement) is the cousin of
  O003-R006's source-conflict shape but scoped to the chain, not
  to status; held local rather than declaring a cross-outcome dep.
- **R002 (Owner Currency).** Owner-side freshness signal. PRD002
  non-transfer flagged explicitly (one timestamp, not two). EC1
  ("never confirmed") explicitly refuses the "confirmed at
  registration" default that would silently erase the gap. R002
  → R001 declared (without an owner, R002 has nothing to time).
- **R003 (Response Window Visibility).** Per-item elapsed signal
  since the bird-dogger's last touch. Reads "last touch" and
  "response received" from R005; R003 → R005 declared.
  Deliberately holds the "touch" definition open (DM, comment,
  mention, meeting note) — same posture as O001-R005 silence-
  window scale or O003-R003 confidence scale. Routine non-
  escalation touches use the same recording shape as escalation
  attempts under R005; the distinction is a label, not a separate
  store. First instance of an R-doc explicitly merging two
  surface concepts into one recording shape.
- **R004 (Escalation Timing Recommendation).** Third
  recommendation-with-override under J001 (O003-R004 status
  override; O004-R003 recommendation override; this). Closed
  three-state enum ("escalate now," "not yet," "cannot
  recommend") for the same reason O004-R003 holds a closed set:
  override needs a stable target. Synthesis signals named per the
  README — slip (O001-R001), intervention window (O004-R003),
  response window (R003), prior history (R005). Five
  dependencies total, four cross-outcome (O005-R001, O005-R003,
  O005-R005 are intra-outcome; O004-R003 and O001-R001 are
  cross-outcome). High dep count but every edge is load-bearing
  per the README's synthesis enumeration.
- **R005 (Escalation Record).** Memory hub for O005. Mitigates
  RSK004. No intra-outcome deps (R003 and R004 read R005, not
  the reverse). Event-log shape declared distinct from
  O004-R004's single-decision shape; "what has changed since"
  framing held off because R005's primary job is preserving the
  attempt log, not anchoring change comparison. EC2 (response
  via different channel than attempt) and EC3 (attempt before
  chain registered) both PRD001 surfacings.

## Boundary calls worth flagging for the judge

- **R005 event-log vs single-decision shape.** I diverged from
  O004-R004's memory-hub shape deliberately — the README wording
  ("each escalation attempt") is plural-event. Judge: accept the
  divergence, or push to model R005 as a current escalation state
  with history derivable from it (which would mirror O004-R004
  more closely)?
- **R003 "touch" definition openness.** Held open and merged
  with escalation attempts at the recording layer (R005 stores
  both). If touches and attempts should live in separate stores
  with different schemas, R003 + R005 both need adjustment. Same
  posture as O004-R004 holding triage-decision vocabulary open.
- **R004 cross-outcome dep on O004-R003 (intervention
  recommendation).** I framed escalation as "the next-tier action
  when intervention is already established as warranted," making
  escalation downstream of intervention. Alternative reading:
  escalation is independent of intervention (an item could
  warrant escalation without first warranting intervention).
  Judge: accept downstream framing, or reframe as parallel
  consumption of shared signals (slip, response window) without
  the tier dependency?
- **No R006 for escalation contact registration.** PRD003
  Consequences explicitly flagged "escalation contact registration
  in O005 is a plausible analogue: duplicate at the requirement
  layer where two outcomes legitimately need it." The README's
  R-list stops at R005, so I did not introduce a R006. Judge or
  reviewer may push to add R006 (parallel to O001-R007 /
  O002-R007 source registration). If added, RSK001 (Misdirected
  Target — "no escalation chain above the owner is recorded")
  would be its mitigator alongside R001.
- **R001 chain disagreement EC vs O003-R006 cousin dep.** R001
  EC3 (chain disagreement across sources) borrows the
  surface-the-conflict posture from O003-R006 but is held local
  to R001 rather than declared as a cross-outcome dep. Same
  judgment call S020 made for R003's override-shape vs.
  O003-R004 (parallel pattern, not consumption). Consistent with
  precedent.
- **R002 non-transfer note on PRD002.** First R-doc to cite a
  PDR to flag where its shape *does not* apply. Reviewer may
  prefer the non-transfer note moved to a CR on PRD002 (clarifying
  PRD002's scope as status-side) rather than carried inline in
  R002. Held inline this session; flag for judge.
- **R004 dep count.** Five deps (3 intra + 2 cross-outcome). High
  relative to other O005 R-docs but every edge is named in the
  README's R004 wording itself ("slip status, intervention
  window, response window"). Same pattern as O004-R002 (five
  cross-outcome edges, accepted by S021 as load-bearing for the
  list-level aggregator role). R004's role here is the synthesis
  hub — analogous load-bearing position.

## What got written

Five files under
`docs/product/outcomes/J001-O005-escalation-quality/requirements/`
(directory created this session — it did not exist; only
`README.md` was in the outcome folder):

- `J001-O005-R001-escalation-target.md`
- `J001-O005-R002-owner-currency.md`
- `J001-O005-R003-response-window-visibility.md`
- `J001-O005-R004-escalation-timing-recommendation.md`
- `J001-O005-R005-escalation-record.md`

All simple template. Each: lifted statement, Detail (with PDR
references inline where the posture applies; with explicit
non-transfer note on R002), Edge Cases (one or more PRD001
surfacings per doc), Examples (one scenario each), Acceptance
Criteria (3 conditions each), Dependencies (intra-outcome plus
named cross-outcome edges).

No Change Record — drafting session in a create cycle (S005 /
S007 / S009 / S011 / S013 / S015 / S017 / S019 / S020 precedent).

## Coverage check

- Every requirement listed in the O005 README has a per-
  requirement document: ✓ (5 of 5).
- Every per-requirement document's statement matches the
  outcome README verbatim: ✓ (trace-preserving).
- Dependency graph: connected DAG, **no cycles**. R001 and R005
  have no intra-outcome deps (hubs); R002 → R001; R003 → R005;
  R004 → R001, R003, R005 + cross-outcome (O004-R003, O001-R001).
  R003 ↔ R005 candidate cycle resolved asymmetrically — R003
  reads R005's recorded touches/responses; R005 does not read
  R003. Same shape as O004 R003 ↔ R004 (S020).
- Cross-outcome dep edges asserted (R004 → O004-R003, O001-R001)
  are consistent with the named outcomes' drafted state.
- Risk–requirement map in the O005 README is unchanged and
  intact: RSK001 → R001, R002; RSK002 → R003, R004; RSK003 → R004;
  RSK004 → R005.
- PDR citations are consistent with PDR Decisions: PRD001 cited
  for "no X recorded" / "delivery failure" surfacings (matches
  Decision); PRD002 cited in R002 to flag shape non-transfer and
  in R003 framing as freshness-cousin (matches Decision, surfaces
  scope honestly); PRD003 not cited (no shared-capability
  requirement under O005 as drafted — flagged in boundary calls).

## Vertical & horizontal trace

- **Vertical:** every requirement traces to a mapped risk; every
  risk impact clause closes on the outcome statement
  ("escalations land with the right person at the right time").
- **Sibling (intra-outcome):** R001 is the chain hub; R005 is
  the memory hub; both stand alone. R002 attaches to R001; R003
  attaches to R005; R004 attaches to R001, R003, R005. Acyclic
  by construction.
- **Cousin (cross-outcome) — new this session:**
  - **R004 consumes O004-R003 (intervention recommendation)** as
    the upstream-tier signal. Frames escalation as the next-tier
    action when intervention is established. See boundary call.
  - **R004 consumes O001-R001 (proactive slip surfacing)** as a
    load-bearing input to timing. Same consumption pattern as
    O004-R003 reading O001-R001.
  - **R002 cousin-of-shape to O003-R002 (status freshness)** —
    parallel pattern, not consumption. Owner-side freshness, one
    timestamp (PRD002 non-transfer noted in-doc). Not in deps;
    flagged in Detail.
  - **R001 EC3 chain-disagreement cousin-of-shape to O003-R006
    (source conflict)** — parallel pattern, not consumption.
    Held local to R001 per S020 precedent on R003 ↔ O003-R004.
  - **R004 override shape cousin-of-pattern to O003-R004 and
    O004-R003** — third instance of override-survives-resynthesis.
    Cited in Detail, not in deps. PDR pressure now real (see
    deferred).
  - **No-owner-at-all seam between O002 and O005** — touched
    this session. O005-R001 EC1 ("no owner known") sits on the
    O005 side of the seam; the O002-side (items without owners
    in coverage) remains where S012 placed it. Seam still
    consistent; not closed but acknowledged in-doc.

## Deferred (so we don't lose it)

New this session:

- **R002 PRD002 non-transfer note** — first R-doc to cite a PDR
  to flag where its shape does not apply. Reviewer may prefer
  this lives as a CR on PRD002 (scoping PRD002 to status-side
  rather than absorbing owner-side as a different shape). Held
  inline.
- **R005 event-log vs single-decision divergence from O004-R004**
  — explicit in Detail. If a judge or reviewer pushes to align
  R005 with O004-R004's single-decision shape, the README's
  R005 wording ("each escalation attempt") would need a parallel
  rewording.
- **R004 escalation-downstream-of-intervention framing** — sees
  O004-R003 as upstream-tier. Alternative parallel framing held
  off. Judge prompt.
- **R003 + R005 touch/attempt merge at the recording layer** —
  one store, label distinguishes touch from attempt. First
  cross-requirement recording-store sharing in J001. Engineering-
  layer assumes one component for both.

Promoted / sharpened PDR-candidates (pressure shape changed
this session):

- **"Override surfacing" as cross-outcome pattern — pressure now
  real.** Flagged in S020, held in S021 (two instances). Third
  instance this session: O003-R004 status override, O004-R003
  recommendation override, O005-R004 escalation timing override —
  all "survives re-synthesis / re-assessment in the same shape,"
  all closed-or-bounded state sets, all PRD001-surfaced on
  "cannot X" cases. Same instance-count shape that drove PRD001
  (3+ instances at outcome layer) and PRD002 (3 R-docs). Strong
  PDR candidate for a session 023 PDR-drafting pass, or for the
  O005 judge to surface.
- **"Cousin-of-shape, not consumption" as a citation pattern** —
  S020 (R003 ↔ O003-R004), S021 (R005 → O002-R006 turned into
  consumption), and this session (R001 EC3 ↔ O003-R006; R002 ↔
  O003-R002 with PRD002 non-transfer; R004 ↔ O003-R004 +
  O004-R003). Four R-docs across two outcomes now use the
  parallel-pattern citation in Detail without a Dependencies-list
  edge. Not yet PDR-shaped (it's a documentation convention, not
  a posture decision), but the convention is now de facto
  precedent.

Carried forward (unchanged or sharpened):

- **PRD001 / PRD002 cited rather than re-derived.** Continued
  working as intended; R-docs are tight. PRD002 use this session
  pushed the cite-pattern slightly — citing to flag non-transfer
  rather than to assert applicability. Honest but new; flagged
  above.
- **PRD003 (shared source registration)** — still not cited under
  O005, but the boundary call on R006 is open. S020 carried the
  "no shared-capability requirement under O004" status forward;
  O005 keeps it pending pending the R006 boundary call.
- **R001/R002/R004 history capability** (audit/event-store
  pressure from O001/O003/O004) — R005 in this session is a
  direct event-log requirement, the sharpest version yet. Fourth
  outcome carrying audit/event-store pressure. Engineering-layer.
- **Override is a multi-requirement shape** — sharpened from
  S017/S018/S020/S021. Now five-to-six requirement shape across
  three outcomes (O003 R001/R002/R004/R005/R006 envelope;
  O004-R003; O005-R004). Component boundary in engineering;
  potentially a fourth PDR per the "Override surfacing" promotion
  above.
- **Engineering concerns sharpened by this session:**
  - **Decisioning component** — R004 joins the synthesis-with-
    override pressure (O001-R001/R005, O002-R001/R005,
    O003-R005/R006, O004-R001/R003). Now spans four outcomes.
  - **Per-item cache with freshness** — R002 (owner-confirmation
    time) and R003 (last-touch elapsed) extend per-item cache.
    Owner-confirmation is a new freshness dimension distinct from
    PRD002's assessed+data-change shape.
  - **History / event store** — R005 is the most explicit event-
    log requirement to date (target, channel, time, response per
    attempt; multiple records per item). Sharpest version of this
    pressure across J001.
  - **Dependency graph store** — unchanged from S020 (O004-R005).
  - **Identity / chain store — new this session.** R001 requires
    a representation of owner + chain-above-owner per item. Cross
    references org structure but is per-item-recorded. ADR
    territory.
- **O001-RSK002 single-mitigation fragility.** Standing from
  S006. Unchanged.
- **No-owner-at-all seam between O002 and O005.** Touched this
  session (R001 EC1) on the O005 side; O002 side unchanged.
- **Missing session 016 (O002 judge).** Standing user decision:
  do not recover, do not re-run. Unchanged.
- **Workspace path mismatch in Cursor.** Eight sessions running
  with the absolute-path workaround.

Cleared this session:

- **None.** S021's six judge prompts were already resolved in
  S021. No standing items closed by this session's drafting.

## Recommended next step

1. **Judge session on the five O005 R-docs.** Likely-fruitful
   prompts (in priority order):
   - **Override surfacing PDR pressure (promoted).** Three
     instances now (O003-R004, O004-R003, O005-R004). Promote
     to PRD004, or hold? Same shape and instance count that
     drove PRD001 and PRD002.
   - R005 event-log shape vs O004-R004 single-decision shape —
     accept divergence (README wording supports), or push to
     align?
   - R004 framing escalation as downstream of intervention via
     O004-R003 dep — accept tier dependency, or reframe as
     parallel consumption of shared signals?
   - R006 escalation contact registration — accept R-list stops
     at 5, or push to add R006 per PRD003 Consequences (shared-
     capability requirement parallel to O001-R007 / O002-R007)?
   - R002 PRD002 non-transfer note — accept inline cite-to-flag,
     or push to a CR on PRD002 that scopes its applicability?
   - R003 + R005 touch/attempt merge at the recording layer —
     accept one store with a label, or push to split?
   - R004 dep count (5 deps including 2 cross-outcome) — accept
     as load-bearing per S021 R002 precedent?
2. **Then commit + PR the five new O005 R-doc files on branch
   `product-docs`** so an independent reviewer can run a review
   session. S021's deferred next-step #1 (review of O004 edits)
   should also land — either as a separate review session or
   bundled with O005 review depending on user preference.
3. **PDR judge pass on PRD001/PRD002/PRD003 — still explicitly
   off the table per S019 user decision.** Do not propose.
   A *new* PRD004 (override surfacing) is a drafting candidate
   under the create cycle, not a judge pass on existing PDRs;
   sequencing depends on user direction in the next session.

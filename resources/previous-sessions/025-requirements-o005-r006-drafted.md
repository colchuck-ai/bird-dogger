# Session 025 — Drafted O005-R006 (Escalation Contact Registration)

## Goal

Draft the R006 that S023 surfaced as standing under O005 and S024
cleared as "queued for a separate session." User came in with "let's
draft the R006" — the queued item per S024's decision-clear. Single
drafting session in a create cycle; no judge pass this session.

## Context loaded

- `assets/templates/requirement-simple.md` from the `intent` skill —
  the template R001–R005 used under O005.
- `intent` skill `SKILL.md` (Requirement rubric + plain language /
  candor guidance), `references/context-tracing.md` (Requirement
  self/siblings/cousins), `references/workflows.md` (create cycle, no
  CR).
- `docs/product/pdrs/PRD003-shared-source-registration.md` — the PDR
  that named R006 as a "plausible analogue" of source registration.
  Sets the shape; R006 picks up the pattern.
- `docs/product/pdrs/PRD004-override-surfacing.md` — checked because
  registering an owner over a source-derived one was a candidate
  PRD004 case; determined it isn't (PRD004 is product-produced
  judgments, not data-population). EC1 cites PRD001 instead.
- `docs/product/pdrs/PRD001-coverage-over-precision.md` — uncertainty
  surfacing posture; referenced from R006 EC1 (owner disagreement)
  and EC4 (chain-depth limit).
- `docs/product/outcomes/J001-O005-escalation-quality/README.md` —
  owning outcome; risks + existing risk-requirement map.
- All five existing O005 R-docs (R001–R005) — for sibling shape,
  vocabulary, citation style, dependency declaration pattern. R001
  loaded as the chain hub R006 produces into; R002 loaded because
  RSK001 also maps to R002 and R006 had to slot alongside it
  cleanly.
- `docs/product/outcomes/J001-O001-slip-detection-lag/requirements/J001-O001-R007-source-registration.md`
  and
  `docs/product/outcomes/J001-O002-coverage/requirements/J001-O002-R007-source-registration.md`
  — the cousin-of-shape R-docs; R006 lifts the statement shape
  ("The product must let the bird-dogger register …"), the Detail
  framing ("registered when the bird-dogger tells the product …"),
  the Edge Cases pattern (unsupported type, duplicate registration),
  and the AC pattern from these.
- `resources/previous-sessions/024-product-pdr004-drafted.md` — the
  decision-clear session; R006 was the "draft as a new requirement
  under O005 in a separate session" item.
- `resources/previous-sessions/023-requirements-o005-judged.md` and
  `resources/previous-sessions/022-requirements-o005-drafted.md` —
  S022 surfaced R006 as a PRD003-flagged analogue; S023 held it as
  out of judge scope and noted the boundary call as open.

Skipped: O003/O004 R-docs (none material to R006). Engineering and
component docs (none exist). Older session digests beyond the
S022–S024 chain (those three carry the R006 thread forward
sufficiently).

## Workspace path mismatch — eleventh session, still standing

Cursor still opens `/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`;
real repo is `/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. S024
recorded the user decision to repoint Cursor; the repoint hasn't
happened yet this session. Workaround unchanged — absolute paths for
all reads + writes through the real repo. Eleven sessions deep now.

**Standing for next session:** unchanged — repoint the Cursor
workspace.

## Scope-shaping decisions (three)

These three calls shape R006 and are surfaceable to the judge /
reviewer. None were AskQuestion-checked with the user before
drafting; the user said "let's draft" and the calls are defensible
on the artifacts that exist.

1. **R006 covers both owner-registration and chain-above-owner.**
   RSK001 names two failure modes — recorded owner is wrong, or no
   chain registered. R001 surfaces the gaps (EC1: "no owner known";
   EC2: "no chain known above owner") but has no mechanism to fill
   them. R006 closes both. Alternative considered: narrow R006 to
   chain-above-owner only, leaving owner-fill out of scope. Rejected
   because R001 EC1 would remain a presentation gap with no
   requirement-layer mitigation. Reviewer may push to split into
   R006 (chain) + R007 (owner) if the dual-scope feels muddled.

2. **Single-outcome, not duplicated.** PRD003 sets the
   shared-capability-shape pattern; the source-registration pair
   (O001-R007 ↔ O002-R007) is the canonical instance. No outcome
   currently besides O005 needs escalation contact registration —
   O001/O002/O003/O004 do not surface ownership or chains in a way
   that requires registering people. So R006 lives only under O005;
   PRD003 is cited for shape inheritance, not for the duplication
   pattern. Reviewer may push to lift R006 to a cross-outcome shape
   or to re-derive the chain-registration shape outside the
   PRD003 frame — both feel like overreach absent a second outcome
   actually needing the capability.

3. **Owner-disagreement is PRD001 territory, not PRD004.** When the
   bird-dogger registers an owner different from the source-derived
   one, the source value isn't a *product-produced judgment* — it's
   data the product reads in. PRD004's scope is synthesized
   recommendations (intervention, escalation timing, status), not
   data-population. EC1 cites PRD001 (uncertainty surfacing) and
   the R001 EC3 multi-source disagreement pattern as the
   analogue. Reviewer may push to either invoke PRD004 (if the
   committee reading of "judgment" is broader) or to add a new
   PRD on data-population disagreement (probably overreach).

## Statement and structure

Statement: "The product must let the bird-dogger register an
escalation contact for an item." Parallels O001-R007 / O002-R007
("register a new data source for tracking items") tightly: action
verb (register), object (escalation contact / data source), scope
anchor (for an item / for tracking items). Single sentence; no
slash-separated alternatives.

Detail follows the R007 pattern:

- **Para 1:** Why R006 exists. Names RSK001's two failure modes;
  identifies R006 as the chain-registration half. Calls out R001's
  presentation gaps explicitly.
- **Para 2:** What R006 names (the *what*). Lists what R006
  doesn't prescribe (engineering-layer punts: identity sourcing,
  chain order, role labels, identity reconciliation).
- **Para 3:** Role of R006 in the risk-requirement map. R006 is
  capability-side; R001 + R002 are presentation-side. Calls out
  the chain-staleness gap (uncovered at requirement layer).
- **Para 4:** Cousin-of-shape to O001-R007 / O002-R007 with PRD003
  citation. Names the single-outcome posture explicitly. Reserves
  PRD003's duplication pattern for future outcomes.

Edge Cases (six):

1. Owner-disagreement with source-derived owner (PRD001 + R001
   EC3 cite).
2. Duplicate registration deduplicated (R007 pattern).
3. Contact without role label registered (R001 role-label punt).
4. Unsupported chain depth surfaced explicitly (PRD001 cite; R007
   "source type not supported" analogue).
5. Chain-member staleness out of scope (R002 only times owner).
6. Registering chain on owner-less item allowed (preserves R001
   EC1 surfacing).

Acceptance Criteria (four): registration without developer
involvement (R007 AC1), registered contact in R001's chain
(verifier for the produce/consume edge to R001), unsupported
shape surfaced (R007 AC3 analogue), owner-disagreement preserves
both visible (covers EC1's PRD001 commitment).

Dependencies: one intra-outcome — R001. R006 produces; R001
consumes. R007's "registered sources enter the refresh cycle"
shape applied to "registered contacts populate the chain R001
surfaces." No cross-outcome dependency declared; the cousin-of-
shape to R007 lives in Detail per the S022/S023 convention.

## What got written

Two files touched:

- `docs/product/outcomes/J001-O005-escalation-quality/requirements/J001-O005-R006-escalation-contact-registration.md`
  — new file, 78 lines, simple template. Statement + Detail (4
  paragraphs) + Edge Cases (6) + Examples (1) + Acceptance Criteria
  (4) + Dependencies (1).
- `docs/product/outcomes/J001-O005-escalation-quality/README.md` —
  added R006 to the Requirements list with summary line synced to
  the R006 statement; added R006 to the RSK001 risk-requirement
  map line.

No PDR drafted. No Change Record — create cycle (workflows.md).

## Boundary calls worth flagging for the judge / reviewer

(Same framing as S022/S024: not held-back judge work; flagged
because R006 was drafted in a single session and the judge pass is
either inline next or in a separate session per the S023 / S024
sequencing decision.)

- **Dual scope (owner + chain) in one R-doc.** Detail and ACs
  cover both; reviewer may push to split or to constrain scope.
- **Single-outcome posture vs PRD003's duplication pattern.** R006
  cites PRD003 for shape, not for duplication. Reviewer may push
  for clearer language on why this case is single-outcome (the
  cousin-of-shape framing is in Detail Para 4).
- **EC1 owner-disagreement citing PRD001 + R001 EC3 rather than
  PRD004.** Reviewer may push for PRD004 invocation if "judgment"
  is read more broadly.
- **EC5 chain-member staleness explicitly out of scope.** A
  reviewer pushing for completeness might ask for a chain-member
  freshness requirement (analogue of R002 lifted from owner to
  chain) — currently flagged as engineering-layer.
- **EC6 chain-on-owner-less-item allowed.** Permissive posture;
  reviewer may push for ordering constraint (owner-first).
- **AC4 owner-disagreement framing.** "Both are visible; the
  source-derived value is not silently dropped" — could be read
  as requiring both to render, or as requiring the source-derived
  to be inspectable. Reviewer may push for sharper language.
- **R006 → R001 sole intra-outcome dep.** Same hub-like shape as
  R001 / R005 (zero or near-zero deps); R006 produces, doesn't
  consume. Reviewer may probe whether R002 should be a declared
  dep (R002 times the registered owner — but R002 already times
  what R001 records, so the chain is R006 → R001 → R002 by
  consumption, no direct R006 → R002 edge needed).

## Coverage check

- Statement follows rubric pattern `[Product/Solution] must
  [Capability/Constraint]`: ✓.
- AC items concrete + testable + solution-free: ✓ (no UI mechanism,
  no implementation specifics; "without developer involvement" is
  the same phrasing R007 uses).
- All ACs verifiable: ✓.
- R006 traces to a mapped risk (RSK001): ✓ (risk-requirement map
  updated in README).
- Statement line in README matches R006 file's statement verbatim:
  ✓.
- Per-doc dependency graph: still a DAG, no cycles. R006 → R001;
  R002 → R001; R003 → R005; R004 → R001/R002/R003/R005 (and
  O004-R003 / O001-R001 cross-outcome); R001 + R005 + R006 are
  intra-outcome hubs (zero intra-outcome deps from them outward
  to deeper requirements; R006 has one *upward* dep to R001).
- PDR citations consistent: ✓ — PRD003 (shape inheritance, not
  duplication); PRD001 (uncertainty surfacing in EC1 and EC4);
  no PRD004 cite (deliberate, per scope-shaping decision 3).
- File-naming convention matches `Files` section: ✓
  (`J001-O005-R006-escalation-contact-registration.md`).
- R-numbering continues from R005: ✓ (R006).

## Vertical & horizontal trace

- **Vertical:** R006 traces to RSK001 (Misdirected Target) under
  O005; RSK001 impact clause closes on the O005 statement
  ("escalations land with the right person at the right time").
  Risk-requirement map updated.
- **Sibling (intra-outcome):**
  - R006 → R001 declared (produces into the chain R001 surfaces).
  - R006 ↔ R002 implicit only — R002 stamps the owner R006
    registers, but the data flow is R006 → R001 → R002 (R002 reads
    R001's record). No direct R006 → R002 dep declared.
  - R006 ↔ R005 no edge. R005 records bird-dogger touches; R006
    records bird-dogger registrations of people. Different stores
    (R005 is event-log of attempts; R006 is identity/chain-list).
    Engineering layer may consolidate stores; requirement layer
    keeps them separate.
- **Cousin (cross-outcome):**
  - R006 cousin-of-shape to O001-R007 and O002-R007 (registration
    pattern). Cited in Detail Para 4 per the S022/S023 convention.
    No declared dependency.
  - No other cousin edges (R006 is single-outcome).
- **PRD horizontal:**
  - R006 → PRD003 (shape inheritance).
  - R006 → PRD001 (in EC1, EC4 — uncertainty surfacing).
  - R006 ↛ PRD004 (deliberate non-cite; data-population is out of
    PRD004's scope).
  - R006 ↛ PRD002 (no freshness presentation concern).

## Deferred (so we don't lose it)

New this session:

- **Judge pass on R006.** Not done this session per user "let's
  draft" framing. Two paths:
  - **Inline judge in a follow-up session, then folded into PR #1
    as additional commits.** Same shape as O005's S022 → S023 split.
    Reviewer can see both the draft and the judge edits in one PR
    diff.
  - **Skip judge, let reviewer pass act as the gate.** Same shape
    as PRD004 (no judge pass per S019 / S024 posture). Reviewer
    flags everything in the boundary-calls section.
  Next session call.

- **PR #1 description doesn't mention R006.** The wholesale PR
  opened in S024 covered through PRD004; R006 is appended. If
  R006 commits to `product-docs`, PR #1's diff will include it but
  the description won't list it under O005. Decision: amend the
  PR description on commit, or leave the boundary-calls list in
  this digest as the reviewer's pointer? S024-style decision-clear.

- **Chain-member staleness.** R006 EC5 names the gap explicitly
  (R002 covers owner only; chain-above-owner staleness uncovered
  at the requirement layer). Could become a future R-doc
  (sharpening R002, or a new R007 under O005) or stay as
  engineering-layer punt. Standing for a future drafting session
  if the reviewer or user pushes.

- **Owner-fill scope inside R006.** R006 covers both owner-fill
  and chain-above-owner. If reviewer pushes to split, R006 becomes
  chain-only and a new R-doc (R007 / R008) covers owner-fill.
  Standing as a possible split.

Promoted / sharpened PDR-candidates:

- **Override-surfacing PDR4** — drafted in S024; no further
  pressure this session. R006 deliberately does *not* invoke
  PRD004 (per scope-shaping decision 3), which the judge / reviewer
  may probe.

Carried forward (unchanged or sharpened):

- **PRD003 (shared-capability pattern)** — R006 cites it but
  doesn't duplicate. First "shape-only" cite of PRD003; sets the
  precedent that PRD003 inheritance can be invoked without invoking
  the duplication-at-requirement-layer pattern when only one
  outcome legitimately needs the capability.
- **Risk map vs Detail-level mitigation roles** (S023, S024
  standing). R006 doesn't add a dual-role complication — it's a
  clean RSK001 direct mitigation. No new pressure on this
  convention.
- **R005 dual role on one store** (S023). Untouched.
- **"Cousin-of-shape, not consumption" as a citation pattern**
  (S022/S023). Reinforced by R006 — Detail Para 4 follows the
  convention.
- **Engineering concerns sharpened by this session:**
  - **Identity / chain store** — first R-doc to write *into* the
    chain (R001 reads from it, R002 stamps it, R005 records
    targets on it). R006's "how identity is reconciled across
    items" engineering-layer punt is the sharpest version of the
    identity-store ADR pressure yet.
  - **Source registration component** — same shape as R006's
    capability layer. PRD003 already calls for one Source
    Registration component shared between O001-R007 and O002-R007.
    R006 may or may not be the same component (it registers
    people, not data sources). ADR question when the engineering
    layer opens.
  - **Override mechanism** — unchanged from S024.
  - **History / event store** — unchanged.
  - **Decisioning component** — unchanged.
  - **Per-item cache** — unchanged.
  - **Dependency graph store** — unchanged.
- **O001-RSK002 single-mitigation fragility** (S006). Unchanged.
- **No-owner-at-all seam between O002 and O005** (S012). R006
  now provides the requirement-layer mechanism for the O005 side
  of the seam to fill the owner from the bird-dogger's knowledge.
  The seam itself (the gap that O002 surfaces as a coverage
  concern vs. O005 surfaces as an escalation-target concern)
  remains; R006 doesn't close it cross-outcome.
- **Missing session 016 (O002 judge).** Standing user decision:
  do not recover, do not re-run. Unchanged.
- **Workspace path mismatch** — eleven sessions running the
  workaround now.

Cleared this session:

- **R006 escalation contact registration drafting candidate
  (S023 standing item, S024 queued)** — drafted. Closed.

## Recommended next step

1. **Decide R006 judge / no-judge.** Mirrors the S024 PRD004
   decision shape:
   - **Inline judge in a follow-up session before reviewer.**
     Same shape as O005's S022 → S023 split. Catches rubric
     defects before the reviewer.
   - **Skip judge; reviewer is the gate.** Same shape PRD004 took
     per S019/S024 posture. Boundary calls in this digest become
     reviewer probes.
   AskQuestion-shaped call.

2. **Commit + push R006 to `product-docs`.** PR #1 is already open
   and tracking that branch; new commits append to the PR. Two
   sub-decisions:
   - Commit message style: short docs-prefix consistent with
     prior commits (`docs: draft j1 o5 r006`).
   - Update PR #1 description to mention R006 under O005? Or
     leave it implicit in the diff and call out only via this
     digest? Reviewer convenience vs. minimal PR-description
     edits.

3. **Reviewer-fruitful probes for R006** (rolled into the O005
   review bundle or split off — see S024 #2):
   - Dual scope (owner + chain) in one R-doc, or split?
   - Single-outcome posture under PRD003, or push for broader?
   - EC1 owner-disagreement via PRD001 / R001 EC3, or push for
     PRD004 invocation?
   - EC5 chain-member staleness as engineering-layer punt, or
     add a chain-freshness R-doc?
   - EC6 chain-on-owner-less item permissive — accept or
     constrain?
   - AC4 owner-disagreement framing — sharpen?
   - R006 → R001 sole dep — accept hub-like shape or add R002?

4. **Workspace path repoint** — user-side action, eleventh
   session standing.

5. **Other still-deferred items unchanged from S024:**
   - O004 review (S021 deferred).
   - O005 review (S023 deferred). R006 now joins the O005 review
     scope.
   - PRD001 / PRD002 back-references to PRD004 (post-merge CR).
   - O004-R004 sharpening to declare open-vocabulary posture
     locally (post-merge CR).

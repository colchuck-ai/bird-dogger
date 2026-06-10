# Session 020 — Drafted requirement documents for O004 Triage Speed

## Goal

Use the `intent` skill to draft the per-requirement documents for
the next important outcome under J001. S019's recommended-next-
step #2 named O004 explicitly; user's opening ask ("draft
requirements for the next important outcome") matched. First R-doc
session after the three product PDRs landed.

## Context loaded

- `docs/product/README.md` — product root; J001 + five outcomes.
- `docs/product/outcomes/J001-O004-triage-speed/README.md` —
  outcome under drafting; statement lines and risk-requirement
  map untouched, lifted verbatim.
- `docs/product/outcomes/J001-O001-slip-detection-lag/README.md`,
  `J001-O002-coverage/README.md`,
  `J001-O003-status-fidelity/README.md` — siblings; read for
  cross-outcome dep targets (R002 → O001-R002, O003-R001/R002;
  R003 → O003-R003; R004 → O002-R006, O003-R002; R005 →
  O002-R001).
- `docs/product/outcomes/J001-O005-escalation-quality/README.md`
  — downstream consumer; read for fit (recommendation override
  shape, freshness consumers) but not internalized in O004
  docs.
- All three PDRs:
  `docs/product/pdrs/PRD001-coverage-over-precision.md`,
  `PRD002-per-item-freshness-presentation.md`,
  `PRD003-shared-source-registration.md` — fully loaded so the
  R-docs could cite rather than re-derive.
- Two sample requirement files from O003 (the most recent
  judged outcome) — `J001-O003-R001-status-basis.md` and
  `J001-O003-R002-status-freshness.md` — for in-doc style +
  verbatim-statement precedent.
- `resources/previous-sessions/017-requirements-o003-drafted.md`,
  `018-requirements-o003-judged.md`,
  `019-product-pdrs-drafted.md` — drafting + judge + PDR
  precedent. S019's recommended-next-step #2 was exactly this
  session.
- `intent` skill: `SKILL.md` (Requirement rubric, PDR
  cross-references), template
  `assets/templates/requirement-simple.md`,
  `references/workflows.md` (create cycle has no CR — confirmed
  from skill, not just precedent). Did not re-read
  `references/context-tracing.md` in full; relied on
  S013/S015/S017 codified decisions.

Skipped: Records (CRs, ADRs) per workflows guidance — drafting
in a create cycle. Did not load O004/O005 outcome digests (S009/
S011) — S017/S018/S019 carried forward the standing flags.

## Workspace path mismatch — still standing, worked around

Sixth session running with the same workaround. Cursor opens
`/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`; real repo
is `/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. Read
tool calls against the workspace path failed; `realpath`
confirmed mismatch on first read. Switched to absolute paths
through the real repo for all reads + writes — same workaround
S017/S018/S019 used. `git status` on `bird-dogger/` shows the
five new R-docs (untracked) plus this digest as the only
changes from this session.

**Standing for next session:** unchanged from S015 onward —
repoint the Cursor workspace at
`/Users/max.dunn/dev/personal/colchuck-ai/bird-dogger`. Six
sessions deep on the workaround now.

## Outcome selection

User asked for "the next important outcome." S019's recommended-
next-step #2 named O004 explicitly. Surfaced the choice via
AskQuestion across three options (O004, O005, both) plus a
second question on PDR-citation style (cite + skip re-derivation,
cite + brief re-statement, no-cite). User selected O004 + cite-
and-skip.

The cite-and-skip choice is the one S019 set up by drafting the
PDRs ahead of O004/O005. First R-doc session to actually use
them this way; format precedent for O005.

## Template choice

Simple template for all 5. None of the O004 requirements carry a
PDR, CR, or other attached material yet (the three product PDRs
are referenced, not nested under O004). Same precedent as O001
(S013), O002 (S015), O003 (S017).

## Requirement framing decisions

Statement lines lifted verbatim from the outcome README's
requirements list — no rewording. Same trace-preservation
pattern as S013 / S015 / S017.

Cross-cutting moves applied to all 5:

- **PDR citations replace re-derivation.** PRD001 cited in
  every R-doc's Edge Cases section (R001 EC1 "cannot rank,"
  R002 EC1 "missing signal," R003 EC1 "cannot recommend," R004
  EC3 "stale decision," R005 EC1+EC3 "cycle / stale
  dependency"). PRD002 cited in R002 (two-timestamp shape at
  list level), R004 ("what has changed" anchored on per-item
  freshness), R005 EC3 (stale dependency). No "coverage over
  precision" posture re-derived; no two-timestamp shape
  re-described.
- **Citations inline in Edge Cases, not in Dependencies.**
  PDRs are decisions, not requirements; treating them as
  dependencies would conflate the two layers. Reviewer may push
  for Dependencies-list citation — first session establishing
  this format, flagged below.
- **Acceptance criteria stay solution-free.** No UI nouns, no
  channel choices, no thresholds, no scale specifications.
  S013/S015/S017 precedent.
- **R-docs shorter than O003.** Cite-and-skip saved roughly the
  PRD001 paragraph from every Detail / Edge Cases section.
  Five R-docs averaged ~75 lines each vs O003's ~85. Smaller
  than predicted in S019 but the right shape — PDR pressure
  was the bulk of the repetition.
- **Edge cases bias toward fail-safe defaults.** Every R-doc
  carries at least one edge case where missing or conflicting
  evidence forces explicit surfacing rather than silent
  omission (PRD001 posture realized per R-doc).

Per-requirement notes:

- **R001 (Urgency Ranking).** Hub for the ordering commitment.
  Granularity scale unspecified at the requirement layer —
  same hold as O003-R003 confidence scale and O001-R005
  silence window. R001 explicitly names "more than two
  buckets" as the granularity floor, which is sharper than
  "differentiates" alone but still solution-free.
- **R002 (List-Level Urgency Signals).** Reads as a presentation
  contract over R001's ranking + cross-outcome signals
  (O001-R002, O003-R001, O003-R002). Two-timestamp shape per
  PRD002 named explicitly in AC #3.
- **R003 (Intervention Recommendation).** Borrows the
  "override survives re-synthesis" shape from O003-R004
  explicitly. Cross-outcome dep on O003-R003 (confidence)
  asserted as load-bearing: low-confidence status must not feed
  a confident recommendation. Edge case 2 cites O003-R004 by
  shape ("same shape as O003-R004").
- **R004 (Triage State Carry-Forward).** Memory hub; R003
  consumes R004 today, R004 reads yesterday's R003 output
  (carry-forward across checkpoints). Acyclic by design — R004
  has no intra-outcome deps; R003 → R004 is the only edge.
  "What has changed" anchored on PRD002 explicitly.
- **R005 (Inter-Item Dependency Surfacing).** Three edge cases
  on cycles, external dependencies, stale dependencies — each
  PRD001 surfacings of "silent handling would mislead triage."
  Direction commitment ("A depends on B" = A blocked by B) is
  explicit; without it the dep graph in the engineering layer
  has no canonical orientation.

## Boundary calls worth flagging for the judge

- **R003 recommendation-level override vs. O003-R004 status
  override.** Two override layers across O003 + O004; R003's
  EC2 borrows O003-R004's "surfaced alongside, not silently
  replaced" shape almost word-for-word. Judge: cross-outcome
  dep, or PDR codifying "override surfacing" as a pattern
  (same family of repetition that drove PRD001/PRD002), or
  accept as local parallelism?
- **R001 "cannot rank" vs R003 "cannot recommend" — two
  PRD001 surfacings.** R001's "cannot rank" feeds R003's
  "cannot recommend." R003 EC1 cites PRD001 only; should it
  cite R001's cannot-rank state directly? I held the indirect
  citation — PRD001 is the canonical anchor.
- **R005 "cycles surface as cycles."** Strong shape claim
  inside an edge case. Could be its own requirement; could
  also be in scope for R005 per the precedent of O003-R006
  ("surface conflict, don't resolve"). Held as edge case.
- **R004 "what has changed" breadth.** Detail asserts the
  recorded context (rank, signals, recommendation) but does
  not enumerate. Verifiable through AC #2 (anchored on PRD002
  shape). Judge may push for tighter enumeration vs. accept
  as engineering-layer.
- **PDR-citation format precedent.** First session to cite
  PDRs inline in Edge Cases (relative paths into `pdrs/`,
  Markdown links). Format is now precedent for O005 unless a
  judge or reviewer changes it. Reviewer may prefer
  Dependencies-list citation; I chose Edge Cases inline so the
  citation sits next to the surfacing it justifies.
- **R002 cross-outcome deps list.** R002 names three
  cross-outcome edges (O001-R002, O003-R001, O003-R002) —
  more than any single R-doc in O001/O002/O003. Each is a
  concrete signal R002 surfaces. Honest but heavy; judge may
  consolidate or accept.
- **R004 cross-outcome dep on O002-R006 reads as consumption,
  not adjacency.** R004 reads refresh-time alongside prior
  triage time. Real consumption dep, same shape as
  O003-R005 → O002-R006. Both consume the refresh cadence.

## What got written

Five files under
`docs/product/outcomes/J001-O004-triage-speed/requirements/`:

- `J001-O004-R001-urgency-ranking.md`
- `J001-O004-R002-list-level-urgency-signals.md`
- `J001-O004-R003-intervention-recommendation.md`
- `J001-O004-R004-triage-state-carry-forward.md`
- `J001-O004-R005-inter-item-dependency-surfacing.md`

All simple template. Each: lifted statement, Detail (with PDR
references inline where the posture applies), Edge Cases (one
or more PRD001 surfacings per doc), Examples (one scenario
each), Acceptance Criteria (3 conditions each), Dependencies
(intra-outcome plus named cross-outcome edges).

No Change Record — drafting session in a create cycle (S005 /
S007 / S009 / S011 / S013 / S015 / S017 / S019 precedent).

## Coverage check

- Every requirement listed in the O004 README has a per-
  requirement document: ✓ (5 of 5).
- Every per-requirement document's statement matches the
  outcome README verbatim: ✓ (trace-preserving).
- Dependency graph: connected DAG, **no cycles**. R001 and R004
  have no intra-outcome deps (hubs); R002 → R001; R003 → R001,
  R004; R005 → R001, R002. R003 ↔ R004 candidate cycle resolved
  asymmetrically — R003 reads R004's prior-cycle output;
  within a single checkpoint R004 has no dep on R003. Improved
  structure over O002's three-cycle draft state (S015) and
  matches O003's clean shape after S018.
- Cross-outcome dep edges asserted (R002 → O001-R002,
  O003-R001, O003-R002; R003 → O003-R003; R004 → O002-R006,
  O003-R002; R005 → O002-R001) are consistent with the named
  outcomes' drafted state.
- Risk–requirement map in the O004 README is unchanged and
  intact: RSK001 → R001, R003; RSK002 → R002; RSK003 → R003;
  RSK004 → R004; RSK005 → R005.
- PDR citations are consistent with PDR Decisions: PRD001 cited
  for "cannot X" surfacings (matches Decision); PRD002 cited
  for per-item freshness presentation (matches Decision);
  PRD003 not cited (no shared-capability requirement in O004 —
  RSK005 dependency surfacing is single-outcome).

## Vertical & horizontal trace

- **Vertical:** every requirement traces to a mapped risk;
  every risk impact clause closes on the outcome statement
  ("identify which items most need intervention right now").
- **Sibling (intra-outcome):** R001 is the ranking hub; R004
  is the memory hub; both stand alone. R002 attaches to R001;
  R003 attaches to R001 + R004; R005 attaches to R001 + R002.
  Acyclic by construction.
- **Cousin (cross-outcome) — new this session:**
  - **R002 surfaces O001-R002 + O003-R001 + O003-R002** as
    list-level signals. PRD002 carries the freshness shape.
  - **R003 consumes O003-R003 (status confidence)** as a
    load-bearing constraint on recommendation strength.
  - **R003 ↔ O003-R004 (status override) — parallel pattern,
    not consumption.** Two override layers, same shape. Not in
    deps; flagged for judge.
  - **R004 consumes O002-R006 (active-set refresh)** as the
    cadence anchor. Same consumption pattern as
    O003-R005 → O002-R006.
  - **R004 consumes O003-R002 (per-item status freshness)** as
    one signal in "what has changed since."
  - **R005 references O002-R001 (coverage set visibility)** as
    the in-set/out-of-set boundary definition.
  - **No-owner-at-all seam between O002 and O005** — unchanged
    from S012/S017/S018/S019; not touched by O004.

## Deferred (so we don't lose it)

New this session:

- **R003 recommendation-level override vs. O003-R004 status
  override** — two override shapes across two outcomes; same
  family of repetition that drove PRD001/PRD002. If O005
  introduces a third override layer (e.g., escalation timing
  override per R004 in that outcome's README — "which the
  bird-dogger can adopt or override"), the PDR pressure is
  three docs deep before any judge runs on O004.
- **PDR-citation format precedent** — Edge Cases inline,
  relative path Markdown links. First session to establish.
  O005 will follow unless changed.
- **R005 cycle-handling commitment** — held as edge case;
  could be lifted to its own requirement if judge pushes.

Promoted / sharpened PDR-candidates (pressure shape changed
this session):

- **"Override surfacing" as cross-outcome pattern.** New
  candidate. O003-R004 (status override survives re-
  assessment), O004-R003 (recommendation override survives
  re-synthesis). If O005-R004 follows the same shape, three
  outcomes carry the pattern. PDR not yet warranted on two
  instances but flagged for sharpening during O005 drafting.

Carried forward (unchanged or sharpened):

- **PRD001 / PRD002 cited rather than re-derived.** First
  session to validate the cite-and-skip pattern. Working as
  intended; R-docs are tighter.
- **PRD003 (shared source registration)** — not cited in any
  O004 R-doc. No shared-capability requirement under O004.
  O005 may have an analogue (escalation contact registration)
  per PRD003 Consequences.
- **R001/R002/R004 history capability** (audit/event store
  pressure from O001 + O003) — R004's record-the-decision-
  with-context extends this. Engineering-layer. Same shape
  S018 carried.
- **Override is a multi-requirement shape** — sharpened from
  S017/S018. O003 had R001/R002/R004/R005/R006 in the override
  envelope; O004 adds R003 (recommendation override). Five-
  to-six requirement shape across two outcomes. Likely a
  component boundary in engineering.
- **Engineering concerns sharpened by this session:**
  - **Decisioning component** — R001 ranking + R003 synthesis
    join the existing decisioning pressure (O001-R001/R005,
    O002-R001/R005, O003-R005/R006).
  - **Per-item cache with freshness** — R002 list-level
    rendering + R004 prior-decision context recorded at
    decision time extend the per-item cache pattern.
  - **History / event store** — R004 carry-forward (decision
    + context per checkpoint) is a direct event-store-shaped
    requirement. Sharpest version of this pressure to date.
  - **Dependency graph store** — new this session. R005
    requires a representation of inter-item dependencies with
    direction, in-set/out-of-set distinction, and cycle
    detection. ADR territory.
- **O001-RSK002 single-mitigation fragility.** Standing from
  S006. Unchanged.
- **No-owner-at-all seam between O002 and O005.** Standing
  from S012. Not touched.
- **Missing session 016 (O002 judge).** Standing user
  decision: do not recover, do not re-run. Unchanged.
- **Workspace path mismatch in Cursor.** Six sessions running
  with the absolute-path workaround.

Cleared this session:

- **PDR-first deferral** — cleared in S019. This session
  validates the choice: R-docs are tighter, PDR references
  carry the posture cleanly, no re-derivation pressure
  surfaced during drafting.

## Recommended next step

1. **Judge session on the five O004 R-docs.** Likely-fruitful
   prompts (in priority order):
   - R003 override-shape vs. O003-R004 — accept parallel-
     pattern framing, or push for a cross-outcome dep, or
     promote to a fourth PDR (override surfacing).
   - PDR-citation format — Edge Cases inline (this session's
     choice) vs. Dependencies-list citation.
   - R002 cross-outcome deps breadth — three edges; accept or
     consolidate.
   - R005 cycle-handling commitment — held as edge case;
     judge may push to lift.
   - R001 "more than two buckets" granularity floor — sharper
     than "differentiates" but still solution-free; judge may
     hold or push for tighter.
   - R004 "what has changed" enumeration — held;
     engineering-layer or requirement-layer?
2. **Then draft O005 R-docs** (Escalation Quality). Five
   requirements per the README: R001 Escalation Target, R002
   Owner Currency, R003 Response Window Visibility, R004
   Escalation Timing Recommendation, R005 Escalation Record.
   R004 in particular is shape-cousin to O004-R003
   (recommendation override) — the "override surfacing" PDR
   pressure will land during O005 drafting if not before.
3. **PDR judge pass on PRD001/PRD002/PRD003 — explicitly off
   the table per S019 user decision.** Do not propose. If
   reviewer feedback on the PRs lands and merits changes,
   those go through update cycles with CRs scoped to the
   changed PDR(s).

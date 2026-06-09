# Session 004 — Judged outcome O003 Status Fidelity

## Goal

Run the judge pass on
`docs/product/outcomes/J001-O003-status-fidelity/README.md` (drafted in
session 003) against the Outcome / Risk / Requirement rubrics in the
`intent` skill, before going to review.

## Context loaded

- `docs/product/outcomes/J001-O003-status-fidelity/README.md` — the
  draft under judgment.
- `docs/product/README.md` — product root for vertical trace.
- `resources/previous-sessions/001-product-root-drafted.md`,
  `002-product-root-judged.md`, `003-outcome-o003-drafted.md` — locked
  decisions, deferred risks, and session 003's standing judge prompts
  (RSK004 distinctness, R005 as lone preventive, R003 testability).
- `resources/bird-dogging/bird-dogging-overview.md` — the practice the
  outcome serves.
- `intent` skill: `SKILL.md` (Outcome / Risk / Requirement rubrics),
  `references/workflows.md`, `references/context-tracing.md`,
  `assets/templates/outcome.md`.

Skipped: Records (CRs, PDRs, ADRs) per workflows guidance — no record
history needed for a judge pass on a create.

## Process notes

- **Vim swap file** `.README.md.swp` present in the outcome folder
  throughout the session. Judged and edited the on-disk version. Author
  should reconcile if vim has unsaved changes.
- **Workspace path mismatch** still standing (sessions 002 / 003): Cursor
  opens `/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`, real repo
  is `/Users/max.dunn/dev/personal/colchuck-ai/bird-dogging-skill`.

## Judge findings

Material (fixed in this session):

1. **RSK002 used "cached status."** Cache is implementation (session 001
   deferred engineering concern). Reframed to "the prior assessment."
   Also renamed RSK002 from "Stale Status Cache" to "Stale Status
   Assessment" to drop the cache framing from the name.
2. **RSK002, RSK003, RSK004 impact clauses didn't reach the outcome.**
   Only RSK001 explicitly tied its impact back to confidence. Tightened
   the other three to close the loop the same way.

Smaller (fixed):

3. **R002 ↔ RSK003 mapping was weak.** R002 shows when tracker data was
   last updated; that catches data that *was* fresh and went stale, not
   a tracker that's been wrong from day zero (the actual RSK003 failure
   mode). Dropped the link.
4. **RSK003 ↔ R001 link added.** Showing inputs and reasoning lets a
   bird-dogger notice "in progress" with no actual evidence — a
   lagging-tracker tell.
5. **R001 "signals or judgments" was abstract.** Reworded to "inputs
   and reasoning" — more concrete for a junior reader.

Resolutions for session 003's standing judge prompts:

- **RSK004 distinctness — keep.** vs RSK001: single-source inference
  error vs multi-source disagreement. vs RSK003: single-source lag vs
  multi-source disagreement. Distinct failure modes.
- **R005 as lone preventive requirement — keep.** Asymmetry is
  intentional and consistent with the product root's "bird-dogger in
  the loop, not autopilot" stance. Flagged: R005 sits on RSK002 alone
  for prevention; if R005 is ever dropped, RSK002 falls back to
  visibility-only mitigation (R002). Worth a PDR if R005 gets
  contested.
- **R003 testability — pass.** R003 promises to *surface* a confidence
  level, which is verifiable. Whether that confidence is *calibrated*
  is a different, harder requirement and not what R003 promises.

Cosmetic, not fixed:

- Outcome statement repeats "status" three times. Lifted verbatim from
  product root, so changing it here would create drift. Leave alone.

## What got edited

`docs/product/outcomes/J001-O003-status-fidelity/README.md`:

- RSK002: renamed "Stale Status Cache" → "Stale Status Assessment";
  body reframed to drop "cached" and to close the impact clause on
  confidence.
- RSK003 and RSK004: impact clauses extended to "...reducing
  confidence that the tracked status reflects reality."
- R001: "signals or judgments" → "inputs and reasoning."
- Risk-Requirement Map row for RSK003: now `R001, R004` (was
  `R002, R004`). Drops the weak R002 link; adds R001.

No Change Record. Create + judge + review form one cycle; revisions
during judge are not a CR.

## Coverage check post-edit

- Every risk has ≥1 requirement: ✓
- Every requirement has ≥1 risk: ✓
- R005 still single-risk (mitigates only RSK002). Standing fragility,
  not a defect.

## Deferred (still standing)

- **RSK003 ↔ O001/O002 boundary.** "Lagging tracker" (here, fidelity
  failure) vs "tracker outage hides slip" (deferred under O001/O002,
  detection failure). Resolve when O001 or O002 are drafted.
- **R005 fragility.** Lone preventive requirement, single-risk
  coverage. Candidate for a PDR if challenged.
- **Engineering concerns** from sessions 001/002: pluggable tracker
  inputs, pluggable cache, pluggable outputs, local-LLM decisioning,
  decision cache keyed by issue-data hash. Attach as components once
  the architecture document exists.
- **Risks under O001/O002** from session 001: tracker outage hides
  slip; new tracker source not integrated yet.
- **Workspace path mismatch** in Cursor.

## Recommended next step

1. Reconcile the vim swap file before committing.
2. Branch + PR these edits (and any uncommitted session-002 product-root
   edits if those are still pending) so an independent reviewer can run
   session 3 against the Outcome / Risk / Requirement rubrics.
3. After review on O003, draft O001 Slip Detection Lag next — that's
   where the RSK003 boundary question gets resolved.

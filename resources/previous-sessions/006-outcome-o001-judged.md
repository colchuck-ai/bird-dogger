# Session 006 — Judged outcome O001 Slip Detection Lag

## Goal

Run the judge pass on
`docs/product/outcomes/J001-O001-slip-detection-lag/README.md` (drafted
in session 005) against the Outcome / Risk / Requirement rubrics in
the `intent` skill, before going to review.

## Context loaded

- `docs/product/outcomes/J001-O001-slip-detection-lag/README.md` —
  the draft under judgment.
- `docs/product/README.md` — product root for vertical trace.
- `docs/product/outcomes/J001-O003-status-fidelity/README.md` —
  sibling outcome, for the resolved RSK003 ↔ O001/O002 boundary.
- `resources/previous-sessions/001` through `005` — locked decisions,
  deferred risks, session 005's standing judge prompts (RSK004
  distinctness, RSK005 fold question, R001 load concentration, R007
  solution-talk pressure, R003 + R004 fold attempt).
- `resources/bird-dogging/bird-dogging-overview.md` — the practice.
- `intent` skill: `SKILL.md` (Outcome / Risk / Requirement rubrics),
  `references/workflows.md`, `references/context-tracing.md`,
  `assets/templates/outcome.md`.

Skipped: Records (CRs, PDRs, ADRs) per workflows guidance — no record
history needed for a judge pass on a create.

## Workspace path mismatch (still standing from sessions 002 / 003 / 004 / 005)

Cursor opens `/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`;
real repo is `/Users/max.dunn/dev/personal/colchuck-ai/
bird-dogging-skill` (resolved via symlink). Worked from the real
path. Worth fixing the workspace mapping.

## Verdict

Clean draft. No material defects. Standing prompts from session 005
all hold. Two minor tightenings applied; both cosmetic.

## Standing judge prompts — resolutions

1. **RSK004 distinctness vs RSK002 — keep distinct.** RSK002 is
   positive signal hidden among healthy items; RSK004 is absence of
   signal indistinguishable from steady progress. Mitigations differ:
   R001 alone covers RSK002, but RSK004 needs R005 because an item
   with no updates does not "appear to be slipping" until you commit
   to interpreting silence as a signal.
2. **RSK005 fold question (outage vs missing integration) — keep
   unified.** Cause differs (transient vs permanent), failure mode is
   identical from the outcome's view: invisible slip. Splitting would
   create the duplicate-failure-mode problem the rubric warns
   against. The cause-vs-mitigation differentiation lives correctly
   inside the requirements (R006 = visibility, R007 = elimination).
3. **R001 load concentration (3 risks) — keep.** R001 is the
   load-bearing capability for the outcome (proactive surfacing),
   parallel to how O003-R001/R002 carry most of that outcome's load.
   Fragility flag: if R001 is decomposed at the architecture stage,
   the decomposition must preserve all three coverage paths
   (RSK001 / RSK002 / RSK004).
4. **R007 solution-talk pressure — keep.** "Register a new data
   source" reads at the right level — *what* the product must accept,
   not *how*. The engineering implication (pluggable inputs at the
   architecture layer) is the right kind of downstream pressure.
5. **R003 + R004 fold attempt — keep separate.** R003 captures the
   expectation; R004 compares against it. Verifiability differs and
   each is broken without the other, but they are distinct
   capabilities.

## RSK003 ↔ O001/O002 boundary — confirmed resolved

The standing deferred item from sessions 003 and 004 stays resolved
as session 005 left it:

- O003-RSK003 (Lagging Tracker Data) — tracker has a value, value is
  wrong → fidelity.
- O001-RSK005 (Source Blind Spot) — tracker has no value at all →
  detection.

No cross-reference added in either README; the IDs and prose are
distinct enough on their own.

## What got edited

`docs/product/outcomes/J001-O001-slip-detection-lag/README.md`:

- **RSK001 reframed.** "The bird-dogger inspects items at intervals;
  slip that begins between inspections is not visible until the next
  inspection..." → "Items are observed at intervals; slip that begins
  between observations is not visible until the next one..." Drops
  the human-perspective pin and lets the risk apply equally to
  bird-dogger inspection cadence and to product polling cadence. R001
  + R002 still mitigate cleanly.
- **RSK005 closing tightened.** Dropped "until it surfaces
  elsewhere" — understated the worst case (slip might never surface
  anywhere if the source is the only one). Now reads "...leaving the
  slip invisible, increasing the time between an item slipping and
  the bird-dogger noticing."

No Change Record. Create + judge + review form one cycle; revisions
during judge are not a CR.

## Coverage check post-edit

- Every risk has ≥1 requirement: ✓
- Every requirement has ≥1 risk: ✓
- All five risk impact clauses close on the outcome statement (`...
  increasing the time between an item slipping and the bird-dogger
  noticing.`). Session 005 preempted the issue session 004 had to
  fix on O003.
- **RSK002 → R001 only** (single-mitigation). Standing fragility,
  parallel to O003-R005's lone-preventive position. Not a defect.

## Deferred (still standing)

- **O001 ↔ O002 boundary** (Slip Detection Lag vs Coverage). Likely
  overlap: items "forgotten between checkpoints" (O002) vs slip
  starting between checks (RSK001 here). Resolves when O002 is
  drafted.
- **R005 (Silence As Signal) interpretation commitment.** PDR
  candidate when "sustained absence" gets operationalized.
- **RSK002 single-mitigation fragility** — new this session, parallel
  to O003-R005. If R001 is ever weakened, RSK002 has no fallback.
- **R005 fragility under O003** (lone preventive requirement,
  single-risk coverage) — still standing from session 004.
- **Engineering concerns** from sessions 001 / 002: pluggable tracker
  inputs, pluggable cache, pluggable outputs, local-LLM decisioning,
  decision cache keyed by issue-data hash. R007 here is the first
  product-level requirement that implies pluggable inputs at the
  engineering layer; attaches as components once the architecture
  document exists.
- **Vim swap file** `.README.md.swp` in the O003 folder — flagged in
  sessions 004 and 005; not touched here. Author should reconcile.
- **Workspace path mismatch** in Cursor.

## Recommended next step

1. Branch + PR these edits (bundling any uncommitted session 002
   product-root edits or session 004 O003 edits if those are still
   pending) so an independent reviewer can run session 7 (review)
   against the same Outcome / Risk / Requirement rubrics.
2. After review on O001, draft **O002 Coverage** next — that's where
   the O001 ↔ O002 boundary question gets resolved, and it leaves
   only O004 Triage Speed and O005 Escalation Quality on the product
   root.

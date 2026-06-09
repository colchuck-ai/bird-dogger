# Session 005 — Drafted outcome O001 Slip Detection Lag

## Goal

Use the `intent` skill to draft the next outcome document under J001.
Picked O001 Slip Detection Lag on every prior session's standing
recommendation (sessions 001, 002, 003, 004 all closed with "draft
O001 next").

## Context loaded

- `docs/product/README.md` — current product root (post session-002
  judge edits).
- `docs/product/outcomes/J001-O003-status-fidelity/README.md` —
  drafted in session 003, judged in session 004. Loaded as the
  sibling outcome that shares boundary territory with O001.
- `resources/previous-sessions/001-product-root-drafted.md`,
  `002-product-root-judged.md`, `003-outcome-o003-drafted.md`,
  `004-outcome-o003-judged.md` — locked decisions, deferred risks,
  and the standing RSK003 ↔ O001/O002 boundary question.
- `resources/bird-dogging/bird-dogging-overview.md` — the practice.
- `intent` skill: `SKILL.md` (Outcome rubric, risk/requirement
  patterns), `references/context-tracing.md` (Outcome vertical /
  horizontal trace), `references/workflows.md` (three-session model),
  `assets/templates/outcome.md`.

Skipped: Records (CRs, PDRs, ADRs) per workflows guidance — no
record history needed for a create.

## Workspace path mismatch (still standing from sessions 002 / 003 / 004)

Cursor opens `/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`;
real repo is `/Users/max.dunn/dev/personal/colchuck-ai/
bird-dogging-skill` (resolved via symlink). Wrote against the real
path. Worth fixing the workspace mapping.

## Outcome selection

Surveyed the four remaining outcomes. O001 won on three grounds:

1. **Chase-loop order.** Outcome ordering is intentional —
   detect → cover → triage → escalate. O001 is step one; without
   trustworthy detection the rest doesn't fire.
2. **Resolves a standing boundary.** Sessions 003 and 004
   explicitly deferred RSK003 ↔ O001/O002 to "when O001 or O002 are
   drafted." This was the moment.
3. **Deferred risks already queued.** Session 001 left two
   O001/O002 risks waiting (tracker outage hides slip; new tracker
   source not integrated). Both land here.

Credible second pick was O002 Coverage — shares the deferred risks
and would resolve the same boundary. Ruled out because O001 is
literally the first chase-loop step; coverage gaps usually surface
*as* detection lag, not the other way around. User confirmed O001.

## Risk framing decisions

Five risks, each customer-centric (bird-dogger's lag, not system's
internals) and solution-free:

- **RSK001 Coarse Check Cadence.** The fundamental lag: any
  interval-based inspection misses slip that starts between checks.
  Self-evident but had to be named — every later requirement keys
  off it.
- **RSK002 Buried Slip Signal.** Signal exists in tracked data but
  is hidden among healthy items. Distinct from "no signal" (RSK005)
  and from "signal absent because owner went quiet" (RSK004).
- **RSK003 No Expected Trajectory.** Without a recorded expectation,
  there's nothing to diverge from — observed progress is just data.
  This is the "you can only detect divergence from an expectation"
  angle.
- **RSK004 Owner Silence.** Negative-space signal: no updates looks
  identical to steady progress. Could fold into RSK002 (buried
  signal) under pressure; I argued distinct because RSK002 is hidden
  positive signal vs RSK004 absence-as-signal.
- **RSK005 Source Blind Spot.** Absorbs both session 001 deferred
  risks ("tracker outage hides slip" + "new tracker source not
  integrated"). Generalized: slip happened in a source the product
  can't read, whatever the trigger. The triggers differ
  (outage vs missing integration) but from the bird-dogger's view
  it's one failure mode. If judge says they're really two, the split
  lands here.

## RSK003 ↔ O003 boundary — resolved

This was the standing deferred item from sessions 003 and 004.
After drafting O001:

- **O003 RSK003 (Lagging Tracker Data)** — tracker has a value; the
  value disagrees with reality. **Fidelity** problem.
- **O001 RSK005 (Source Blind Spot)** — tracker has no value at all
  (outage or no integration). **Detection** problem.

Triggers can overlap. Impacts are clean and the risks sit cleanly
under their respective outcomes. No cross-reference added in either
README; the IDs and prose are distinct enough on their own.

## Requirement framing decisions

Seven requirements, each tied to ≥1 risk. All behavioral, not
implementation:

- **R001 Proactive Slip Surfacing** is the load-bearer (3 risks:
  RSK001, RSK002, RSK004). Symmetric with O003's R001/R002 carrying
  most of the load.
- **R002 Assessment Freshness** is single-risk (RSK001 only).
  Companion to R001 — surfacing items is useless if the bird-dogger
  doesn't know how fresh the picture is.
- **R003 Expected Trajectory** and **R004 Trajectory Comparison**
  are kept as two requirements. One is data capture (record the
  expectation), one is detection (compare against it). Justified as
  separate capabilities; judge may try to fold.
- **R005 Silence As Signal** is the lone *interpretation*
  requirement — most other R's surface or compare data; R005 commits
  the product to a specific interpretation rule (silence ≠ progress).
  Distinctive. PDR candidate if challenged.
- **R006 Source Availability** and **R007 Source Registration** both
  mitigate RSK005 from different angles. R006 makes the blind spot
  visible; R007 lets the bird-dogger eliminate it. R007 borders on
  solution-talk ("register a new source") but the *what* is product;
  *how* it integrates is engineering. Watch for judge pushback.

Risk-Requirement Map:

- RSK001 → R001, R002
- RSK002 → R001
- RSK003 → R003, R004
- RSK004 → R001, R005
- RSK005 → R006, R007

## What got written

`docs/product/outcomes/J001-O001-slip-detection-lag/README.md` —
full outcome document with the outcome statement (lifted verbatim
from the product root for narrative coherence, same move as session
003), five risks, seven requirements, and the risk-requirement map.

No Change Record — this is a create, per workflows.md.

## Coverage check

- Every risk has ≥1 requirement: ✓
- Every requirement has ≥1 risk: ✓
- RSK002 has single-requirement coverage (R001 only). Standing
  fragility, parallel to O003 R005. If R001 ever weakens, RSK002 has
  no fallback. Worth flagging in judge.
- R001 carries 3 risks. Load concentration is intentional but worth
  defending in judge.

## Deferred (so we don't lose it)

- **O001 ↔ O002 boundary.** Slip Detection Lag vs Coverage. Did not
  pre-resolve. Lands when O002 is drafted. Likely overlap: items
  "forgotten between checkpoints" (O002) versus slip that began
  between checks (RSK001 here). The framings touch but don't
  collapse; they should sort themselves out when O002's risks get
  named.
- **RSK004 vs RSK002 distinctness.** Argued distinct (absence-as-
  signal vs hidden positive signal). Judge should test.
- **R007 (Source Registration) solution-talk pressure.** May get
  pushed back as engineering.
- **R005 (Silence As Signal) interpretation commitment.** PDR
  candidate if challenged on what "sustained absence" means.
- **R003 + R004 fold-risk.** Capture-vs-compare may face a fold
  attempt; defended as two distinct capabilities.
- **Engineering concerns** from sessions 001 / 002 still standing:
  pluggable tracker inputs, pluggable cache, pluggable outputs,
  local-LLM decisioning, decision cache keyed by issue-data hash.
  R007 here is the first product-level requirement that *implies*
  pluggable inputs at the engineering layer; attaches as components
  once the architecture document exists.
- **R005 fragility under O003** (lone preventive requirement,
  single-risk coverage). Still standing from session 004.
- **Vim swap file** `.README.md.swp` in the O003 folder — session
  004 flagged it; not touched here. Author should reconcile.
- **Workspace path mismatch** in Cursor.

## Recommended next step

1. Branch + PR these changes so an independent reviewer can run
   session 6 (judge pass) against the Outcome / Risk / Requirement
   rubrics. Bundle in any uncommitted session-002 product-root edits
   or session-004 O003 edits if those are still pending.
2. Run the judge pass on
   `docs/product/outcomes/J001-O001-slip-detection-lag/README.md`.
   Likely-fruitful prompts:
   - RSK004 distinctness (vs RSK002).
   - RSK005 single-risk fold question (should outage and missing-
     integration split into two risks?).
   - R001 load concentration (three risks on one requirement).
   - R007 solution-talk pressure.
   - R003 + R004 fold attempt.
3. After judge + review on O001, draft O002 Coverage next — that's
   where the O001 ↔ O002 boundary question gets resolved, and it
   leaves only O004 Triage Speed and O005 Escalation Quality on the
   product root.

# Session 003 — Drafted outcome O003 Status Fidelity

## Goal

Use the `intent` skill to draft the next outcome document under J001.
Picked O003 Status Fidelity on both prior sessions' standing
recommendation.

## Context loaded

- `docs/product/README.md` — current product root (post session-002
  judge edits).
- `resources/previous-sessions/001-product-root-drafted.md` and
  `002-product-root-judged.md` — locked decisions, deferred risks,
  and the standing O003-first recommendation.
- `resources/bird-dogging/bird-dogging-overview.md` — the practice.
- `intent` skill: `SKILL.md` (Outcome rubric, risk/requirement
  patterns), `references/context-tracing.md` (Outcome vertical /
  horizontal trace), `references/workflows.md` (three-session model),
  `assets/templates/outcome.md`.

Skipped: Records (CRs, PDRs, ADRs) per workflows guidance — no record
history needed for a create.

## Outcome selection

Surveyed all five outcomes. O003 won on downstream leverage: its
deferred risks (LLM mis-judges, stale decision cache, tracker data
wrong/lagging) attach to most of session 001's deferred engineering
pluggability. Named O001 Slip Detection Lag as the credible second
choice (first step of chase loop; O002 / O004 lean on O003's data
being trustworthy). User confirmed O003.

## Workspace path mismatch (still standing from session 002)

Cursor opens `/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`;
real repo is `/Users/max.dunn/dev/personal/colchuck-ai/
bird-dogging-skill` (resolved via symlink). Wrote against the real
path. Worth fixing the workspace mapping.

## Risk framing decisions

Took the three deferred O003 risks from sessions 001 / 002 and
reframed each customer-centric and solution-free:

- "LLM mis-judges status" → **RSK001 Bad Status Inference**.
  Dropped the LLM-specific framing; the failure mode is *any*
  automated interpretation disagreeing with truth.
- "Stale decision cache after tracker schema change" →
  **RSK002 Stale Status Cache**. Generalized: the underlying data
  changing (schema or content) is one trigger; staleness is the
  failure.
- "Tracker data wrong/lagging" → **RSK003 Lagging Tracker Data**.
  Kept tightly scoped to "tracker not updated to match reality."

## RSK004 added (flag for judge)

Added **RSK004 Conflicting Signals** — sources disagree, bird-dogger
can't tell which to trust. Not in either prior session's deferred
list. Justification: pluggable-tracker framing implies multi-source,
and "no single signal is wrong, but they don't agree" is a distinct
failure mode from RSK001 / RSK003. Judge pass should challenge
whether it actually is distinct.

## Requirement framing decisions

Six requirements, each tied to ≥1 risk. All behavioral, not
implementation:

- **R001 Status Basis** and **R002 Status Freshness** carry most of
  the load — each maps to two risks.
- **R003 Status Confidence**, **R004 Status Override**, **R006
  Conflict Surfacing** are single-risk requirements that add
  specific affordances.
- **R005 Re-Assessment On Change** is the only *preventive*
  requirement; the other five make problems visible or recoverable
  rather than preventing them. Matches the product root's
  "bird-dogger in the loop, not autopilot" stance — worth checking
  in judge.

Risk-Requirement Map:

- RSK001 → R001, R003, R004
- RSK002 → R002, R005
- RSK003 → R002, R004
- RSK004 → R001, R006

## What got written

`docs/product/outcomes/J001-O003-status-fidelity/README.md` — full
outcome document with the outcome statement (lifted verbatim from
the product root for narrative coherence), four risks, six
requirements, and the risk-requirement map.

No Change Record — this is a create, per workflows.md.

## Deferred (so we don't lose it)

- **RSK003 ↔ O001/O002 boundary.** "Lagging tracker" here vs.
  "tracker outage hides slip" in the O001/O002 deferred risks.
  Different framings of related conditions. Resolve when O001 or
  O002 docs get drafted.
- **RSK004 challenge.** May fold into RSK001 or RSK003 under
  judge pressure.
- **Engineering concerns from sessions 001/002** still standing:
  pluggable tracker inputs, pluggable cache, pluggable outputs,
  local-LLM decisioning, decision cache keyed by issue-data hash.
  These attach as components once the architecture document
  exists.
- **Risks under O001/O002** from session 001: tracker outage hides
  slip; new tracker source not integrated yet.

## Recommended next step

1. Branch + PR these changes (the session-002 judge edits to the
   product root are still uncommitted; bundle them in or commit
   separately — author's call).
2. Run the judge pass on `docs/product/outcomes/
   J001-O003-status-fidelity/README.md` against the Outcome rubric.
   Likely-fruitful prompts for judge: RSK004 distinctness; R005 as
   the lone preventive requirement; whether R003 (confidence) is
   testable enough.
3. After judge + review on O003, draft O001 Slip Detection Lag
   next — that's where the RSK003 boundary question gets resolved.

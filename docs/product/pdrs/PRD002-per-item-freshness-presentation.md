# PRD002 - Per-Item Freshness Presentation

Per-item freshness is shown as two distinct timestamps — when the
product last assessed the item, and when its underlying data last
changed — kept distinct rather than collapsed.

## Context

Three R-docs already carry a freshness presentation, and two of
them share the same per-item shape:

- **O001-R002 (Assessment Freshness).** Per-item "last assessed"
  timestamp surfaced alongside the item's slip reading so a
  "looks fine" answer cannot rest on a stale assessment.
- **O003-R002 (Status Freshness).** Per-item "last assessed" plus
  "last underlying data change," distinct, surfaced together. The
  gap between them is the staleness signal — assessment recent
  but data old means the assessment still holds; data new but
  assessment old means something has moved since the last look.
- **O002-R006 (Active-Set Freshness).** Set-level "last refresh"
  timestamp on the active set. Different scope (set, not item)
  but same family of "show the freshness explicitly" thinking.

The two-timestamp shape (assessed + underlying changed) drives
**O003-R005 (Re-Assessment On Change)**: the gap is the trigger
condition. Without a recorded decision, a future R-doc could
collapse the two into a single "last updated" and quietly break
the staleness signal that downstream requirements rely on.

O004 R-docs that synthesize urgency (R002 List-Level Urgency
Signals, R004 Triage State Carry-Forward) and any future O005
R-doc that drives escalation timing will need freshness alongside
their list views — the fourth and fifth instances if drafted
without a recorded shape. Pressure flagged across S015, S017, and
S018.

## Options

- **A. Two-timestamp per-item presentation.** "Last assessed" and
  "last underlying data change" surfaced distinctly per item, both
  visible at the list level. The picking rule for multi-source
  underlying-data values is engineering-layer (already noted in
  O003-R002). Set-level freshness (e.g., O002-R006) is a separate
  cousin shape under the same posture and is not collapsed into
  per-item.
- **B. Single-timestamp ("last touched") per item.** Collapse the
  two into one rolled-up freshness. Cost: loses the gap signal —
  the gap is what tells the bird-dogger whether to re-read.
  O003-R005 directly relies on the gap; collapsing it requires
  re-deriving the trigger from somewhere else.
- **C. Per-item-only or set-level-only.** Pick one scope and force
  the other to derive from it. Cost: per-item alone misses
  set-level coverage gaps (O002-R006); set-level alone misses
  per-item drift (O001-R002, O003-R002). Both scopes answer real
  questions.

## Decision

Option A — two-timestamp per-item presentation.

The two timestamps answer two different questions: "when did we
last look?" and "when did the world last change?" The gap between
them is the actionable signal, and downstream requirements
(O003-R005 re-assessment, O004 urgency synthesis) consume the gap,
not a single number. The set-level / per-item split stays — they
answer different questions and the bird-dogger needs both.

## Consequences

- O004 R-docs that surface freshness alongside urgency can
  reference this PDR for the shape rather than re-deriving the
  two-timestamp framing. Same for any future O005 R-doc that
  needs freshness in escalation timing.
- One freshness contract for the engineering layer: a per-item
  store with two timestamps plus a set-level refresh marker, not
  three subtly-different contracts.
- Rendering two timestamps per item costs more list-level space
  than a single "last touched"; the bird-dogger needs to learn
  the distinction. R-docs that surface freshness must keep both
  readable, not bury the data-change time.
- When a source does not expose an underlying-data-change time,
  the data-change timestamp surfaces as "unknown" per
  [PRD001](PRD001-coverage-over-precision.md). The two PDRs
  reinforce each other — coverage-over-precision is the posture;
  the two-timestamp shape is one place the posture is realized.
- The engineering layer owns the picking rule for multi-source
  underlying-data timestamps (max-of, per-source, composite).
  This PDR commits to surfacing one per-item value. The picking
  rule is ADR territory.

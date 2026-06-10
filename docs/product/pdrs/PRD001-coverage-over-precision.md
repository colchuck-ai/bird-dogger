# PRD001 - Coverage Over Precision

When the product cannot judge an item's state with confidence, it
surfaces the gap explicitly rather than defaulting to a confident
reading.

## Context

Across O001, O002, and O003 R-docs, edge cases consistently force
the same answer: when evidence is missing, stale, contradictory, or
unreachable, the product makes the gap visible rather than picking
a default that hides it.

Concrete instances already in the requirement layer:

- **O001-R001** surfaces "cannot assess" as a distinct state for
  items with insufficient slip signal; never silently treats them
  as healthy.
- **O001-R002** surfaces "not yet assessed" rather than blank or
  zero.
- **O001-R004** produces "cannot assess from trajectory" rather
  than a misleading on-track reading when no trajectory is
  recorded.
- **O001-R006** surfaces source unavailability per item rather
  than letting silence read as on-track.
- **O002-R001** shows an empty watched-sources view explicitly as
  empty, not absent.
- **O002-R004** surfaces drop-offs as a distinct set rather than
  silently letting items leave coverage.
- **O002-R006** flags unreachable sources at refresh time and
  retains their prior items as stale rather than silently dropping
  them.
- **O003-R002** surfaces "unknown data-update time" explicitly
  rather than blanking or assuming.
- **O003-R005** surfaces stale assessments when re-assessment is
  deferred rather than holding the prior reading as fresh.
- **O003-R006** surfaces conflict between sources rather than
  silently picking one.

Each R-doc currently re-derives this posture in its own Detail or
Edge Cases section. Without a recorded decision, every new
requirement in O004 / O005 will repeat the derivation, drift on
wording, or get prompted into a different posture by review
pressure. Pressure has been flagged across S014, S015, S017, and
S018.

## Options

- **A. Coverage over precision.** When evidence is missing, stale,
  contradictory, or unreachable, the product surfaces the gap
  explicitly. Items in those states stay in coverage and are shown
  with the right "we don't know" or "evidence disagrees" label
  rather than rolled into a confident bucket.
  Cost: more states to render and explain; the bird-dogger sees
  more "uncertain" readings, which can feel noisy at a glance.
- **B. Precision over coverage.** The product picks the most
  likely confident reading when evidence is incomplete (e.g.,
  default-to-on-track, single-source-wins, prior-reading-holds).
  Cost: the bird-dogger cannot tell when a reading is real vs.
  asserted by default; a chase loop running on quiet defaults
  misses the cases the product exists to surface.
- **C. Mixed (per-outcome).** Each outcome owns its own posture
  choice. Cost: the bird-dogger needs different mental models per
  outcome; cross-outcome states (e.g., status "cannot assess"
  feeding triage urgency feeding escalation timing) compose badly
  when the rule changes per layer.

## Decision

Option A — coverage over precision.

The bird-dogging job explicitly fails when items fall through
silently; the J001 outcomes are framed around closing those silent
paths (slip detection lag, coverage gaps, status fidelity). A
precision-first posture undercuts the whole job. The mixed posture
composes badly because uncertainty propagates across outcomes —
status confidence feeds urgency, which feeds escalation timing —
and a different rule per layer breaks the chain.

## Consequences

- Per-requirement docs in O004, O005, and any later outcome can
  reference this PDR rather than re-deriving the posture in each
  Edge Cases section. Same for new R-docs added under existing
  outcomes via update cycles. Reduces drift risk between sibling
  requirements.
- Gives the engineering layer a single posture to satisfy when
  designing the per-item state model — every item state must
  include explicit room for "we don't know" / "evidence disagrees"
  rather than collapsing to a binary healthy/unhealthy.
- The bird-dogger's surface area grows: more distinct states than
  the precision posture would render. R-docs and downstream UI
  patterns must keep these states recognizable at a glance, not
  bury them.
- Judges and reviewers should now pull on requirements that *do
  not* exhibit this posture, not requirements that do. R-docs
  still name the specific surfacing they own (e.g.,
  "stale-assessment surfaced," "source-unavailable surfaced") —
  this PDR sets the posture, not the wording.
- When a future requirement legitimately needs a precision-first
  read (the bird-dogger explicitly asks "what's the best guess?"),
  it must call out the deviation and explain why precision is
  correct for that case. Default is coverage.

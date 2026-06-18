# PRD003 - Shared Source Registration

Source registration is one capability that serves multiple
outcomes. The requirement appears under each outcome that needs
it; the implementation is shared.

## Context

**O001-R007** and **O002-R007** are word-for-word identical:

> The product must let the bird-dogger register a new data source
> for tracking items.

Both R-docs' Detail sections explicitly note the duplication is
intentional and flag it for engineering-layer consolidation. The
risk-requirement maps point at them from different risks:

- **O001-RSK005 (Source Blind Spot)** — slip in a source the
  product cannot read is invisible. R007 mitigates by letting the
  bird-dogger register the source.
- **O002-RSK002 (Unwatched Source)** — items in unregistered
  sources are outside every checkpoint. R007 mitigates by adding
  the source to coverage.

Without a recorded decision, future judges may push to deduplicate
at the requirement layer — collapse one R007 into the other, or
lift it to product-level — and lose the per-outcome trace that
each outcome's local read relies on.

## Options

- **A. Shared capability, duplicated requirement.** Keep R007
  under each outcome that needs it. Each R-doc names the
  duplication in its Detail and points at the other outcome's
  R007. Engineering-layer consolidates the implementation; the
  requirement layer keeps per-outcome trace intact.
- **B. Single-home requirement.** Pick one outcome to own R007
  (e.g., O002, where coverage setup is the natural home), and
  have other outcomes depend on it cross-outcome. Cost: breaks
  per-outcome trace — O001's risk-requirement map for RSK005
  would point at an O002 requirement, complicating the local
  read of O001.
- **C. Promote to product-level.** Lift source registration out
  of any single outcome and treat it as a product-wide capability
  declared at the product README level. Cost: introduces a
  product-level requirement layer with no precedent in the
  current docs; over-engineers a single shared capability.

## Decision

Option A — shared capability, duplicated requirement.

The duplication is honest about the fact that both outcomes
legitimately need the capability. Keeping each outcome's
risk-requirement map self-contained preserves the local read
clarity the framework's vertical trace depends on. The
engineering-layer consolidation (one component, one
implementation) is captured separately and does not require
collapsing requirement-layer trace.

## Consequences

- Each outcome README stays self-contained: O001-RSK005 and
  O002-RSK002 both map to a R007 that lives under their own
  outcome.
- Sets the pattern for any future shared-capability requirement.
  No current candidate is identified in O004 or O005, but
  escalation contact registration in O005 is a plausible
  analogue: duplicate at the requirement layer where two outcomes
  legitimately need it, share at the component layer.
- Maintaining two R-docs costs double the edit surface — any
  change to source registration scope must update both files.
  Drift risk is real if a future update only edits one.
- Judge sessions on either O001-R007 or O002-R007 should always
  check the other for parallel updates. The next judge rubric
  pass should add a standing prompt to that effect.
- When the engineering layer lands, it should commit to a single
  Source Registration component (not one per outcome). ADR
  territory; this PDR sets the requirement-layer expectation,
  not the component layout.
- Future de-registration capability — flagged in O001-R006 EC3 —
  follows the same pattern when it is drafted: shared capability,
  duplicated requirement under each outcome that needs it.

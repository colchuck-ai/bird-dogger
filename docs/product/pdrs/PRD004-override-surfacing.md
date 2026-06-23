# PRD004 - Override Surfacing

When the bird-dogger overrides a product-produced judgment, the override stands across re-synthesis; new synthesis output is surfaced alongside the override for adoption or dismissal, never as a silent replacement.

## Context

Three R-docs across three outcomes carry the same override shape:

- **O003-R004 (Status Override).** The bird-dogger can override an item's tracked status. R005's re-assessment must not silently replace the override; when the inferred basis disagrees, the disagreement is surfaced alongside the override for the bird-dogger to adopt or dismiss. Override is first-class in the basis (R001 traces it) and in the freshness story (R002 timestamps it as the last assessment).
- **O004-R003 (Intervention Recommendation).** The product synthesizes an "intervene now" / "not now" / "cannot recommend" call. The bird-dogger can override; R003's Detail explicitly commits to "the same shape O003-R004 commits to for status" — override survives re-synthesis, disagreement surfaces alongside.
- **O005-R004 (Escalation Timing Recommendation).** The product synthesizes an "escalate now" / "not yet" / "cannot recommend" call. The bird-dogger can override; R004's Detail explicitly commits to "the same shape O003-R004 and O004-R003 commit to."

These three are distinct overrides at distinct levels — status, intervention recommendation, escalation timing recommendation — not one override repeated. Each R-doc currently re-derives the behavior in its Detail and Edge Cases section. Cross-R-doc references are explicit ("same shape as O003-R004") but the shape itself lives nowhere central — every new requirement that produces a per-item judgment will repeat the derivation, drift on wording, or get prompted into a different posture by review pressure.

The shape carries two coupled commitments:

1. **Override survives re-synthesis.** New synthesis output never silently replaces an active override. Both are surfaced, and the bird-dogger adopts the new call or dismisses it.
2. **Closed-state vocabulary on the produced judgment.** The override needs a stable target, and list-level rendering must show a known state. O004-R003 and O005-R004 carry explicit three-state enums for this reason; O003-R004's status vocabulary is drawn from the basis (R001), not free text.

## Options

- **A. Persistent override with surfaced disagreement.** The bird-dogger's override is a first-class state on the item, kept as the displayed value across re-synthesis. When new synthesis produces a different call, both are surfaced together; the bird-dogger adopts or dismisses. The product's produced judgment stays a closed-vocabulary state so the override has a stable target. Cost: more states to render (override + new reading + disagreement marker); more state to maintain (override is persistent until cleared, not derivable from current signals).
- **B. Override expires on re-synthesis.** Each re-synthesis pass recomputes the call from current signals; a prior override is cleared so the freshest reading always wins. Cost: bird-dogger knowledge the product cannot infer (owner conversation, side-channel update, observed shipped artifact) gets silently overwritten — exactly the case the override exists to handle. The bird-dogger has to re-apply the override every checkpoint.
- **C. Mixed (per-outcome).** Each outcome owns its own override-survival posture. Cost: O003-R004 already commits to persistence; O004-R003 and O005-R004 explicitly cite that shape forward. Diverging at any of the three breaks the chain — a status override that survives re-assessment but an intervention override that does not would let a later synthesis pass re-derive "intervene now" from a status the bird-dogger had explicitly overridden. Composition fails the same way mixed posture fails for [PRD001](PRD001-coverage-over-precision.md).

## Decision

Option A — persistent override with surfaced disagreement.

The bird-dogger overrides the product when they have direct knowledge the product cannot infer — owner conversation, side-channel update, observed shipped artifact, judgment that weighs context the synthesis rule does not capture. Letting re-synthesis quietly overwrite that authority defeats the override. Surfacing disagreement preserves both the bird-dogger's call and the product's fresh reading; the bird-dogger decides which to keep. The closed-state vocabulary is the constraint that lets the override have a stable target and lets list-level rendering remain readable across re-synthesis cycles.

Per-outcome divergence (Option C) is rejected for the same composition reason coverage-over-precision is one posture across outcomes: overrides at one layer feed inputs at another (status feeds urgency feeds escalation timing), and a layer that silently replaces an override breaks the layers above it.

## Consequences

- Per-requirement docs in O004, O005, and any later outcome that introduces a product-produced judgment can reference this PDR rather than re-deriving the override-survives-resynthesis shape in each Detail and Edge Cases section. Same for new R-docs added under existing outcomes via update cycles. Reduces drift risk between sibling overrides.
- Any future requirement that produces an overridable per-item judgment must declare the closed-state vocabulary its override targets. Free-text or open-vocabulary judgments are not under this PDR's scope and need an explicit deviation if drafted — e.g., O004-R004 (Triage State Carry-Forward), where the bird-dogger supplies the label and the workflow owns the vocabulary, is intentionally outside this shape.
- The bird-dogger's surface area carries the override as a first-class state distinct from "the product's current call," with a visible disagreement marker when the two differ. R-docs surfacing overridable judgments must keep the disagreement marker readable — same readability obligation [PRD001](PRD001-coverage-over-precision.md) imposes for uncertainty states.
- Override clearing returns the displayed value to the current synthesis output; the cleared override remains recoverable in history (basis trace for status under O003-R001; equivalent trace for the recommendation-layer overrides). R-docs in this pattern should include an explicit "cleared override remains recoverable" acceptance criterion.
- This PDR composes with [PRD001](PRD001-coverage-over-precision.md): when synthesis cannot produce a confident call, the produced state is "cannot recommend" (or the outcome's analogue) per PRD001; when the bird-dogger overrides, the override stands per PRD004. The two postures together cover both the bird-dogger's uncertainty and the bird-dogger's authority — neither is silently overwritten by the other.
- Engineering layer carries one override mechanism, not three: per-item override field with adopt/dismiss affordance and a disagreement-surfacing renderer, parameterized by which product-produced judgment the override covers. ADR territory when the engineering layer lands; this PDR sets the requirement-layer expectation, not the component layout.
- Judges and reviewers should now pull on requirements that produce overridable judgments without exhibiting this shape (silent replacement, free-text produced state, missing disagreement surfacing), not requirements that do. R-docs still name the specific override they own (e.g., status, intervention, escalation timing) — this PDR sets the shape, not the wording.

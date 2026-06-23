# J001-O004-R003 - Intervention Recommendation

The product must synthesize the urgency signals into a recommendation of which items need intervention right now, which the bird-dogger can adopt or override, rather than leaving the synthesis entirely to the bird-dogger each checkpoint.

## Detail

R001 ranks; R002 surfaces; R003 calls it. The product makes the "intervene now" judgment so the bird-dogger does not re-derive the threshold from raw signals every checkpoint (RSK003), and does not have to read down R001's ranking item-by-item to pick the next target (the evaluation cost in RSK001).

R003 names the *what*: a per-item recommendation of "intervene now" or "not now," derivable from the urgency signals and the prior triage state carried by R004. The bird-dogger can adopt or override it. The override is recommendation-level — distinct from R001's ranking and from O003-R004's status override — and survives re-synthesis on later checkpoints in the same shape O003-R004 commits to for status.

R003 does not prescribe the synthesis rule (weighting, threshold, model). The threshold is engineering-layer; R003 commits to producing a recommendation that traces to the signals it weighed.

The three recommendation states ("intervene now," "not now," "cannot recommend") are a closed set: the override needs a stable target and the list-level rendering under R002 must show a known state, not a free-text call. This differs from R004, where the decision vocabulary is intentionally open — under R004 the bird-dogger supplies the label and the workflow owns the vocabulary; under R003 the product produces the call and a closed enum is what the override and the renderers can rely on.

## Edge Cases

- Signals insufficient to make a recommendation (item in R001's "cannot rank" state, status confidence too low per O003-R003): surfaced as "cannot recommend" per [PRD001](../../../pdrs/PRD001-coverage-over-precision.md), not defaulted to "no intervention needed."
- Bird-dogger has overridden the recommendation on a prior checkpoint and the next synthesis pass produces a different call: the override is not silently replaced; the new synthesis output is surfaced alongside the override for the bird-dogger to adopt or dismiss. Same shape as O003-R004.
- Recommendation flips between checkpoints (last checkpoint said "not now," this checkpoint says "intervene now"): the flip is visible as a transition against the prior triage state from R004, not silently replacing it.

## Examples

### Recommendation on an item with low-confidence status

- Input: item with red slip signal but low status confidence (O003-R003) because sources disagree.
- Expected Output: R003 surfaces "cannot recommend" rather than a high-confidence "intervene now" — the low-confidence status feeds forward, not silently laundered into a confident recommendation. The bird-dogger can adopt or override.
- Verification: a checkpoint review on the item shows the recommendation state alongside the conflict, not a confident call independent of the confidence signal.

## Acceptance Criteria

- [ ] Each item carries an explicit "intervene now," "not now," or "cannot recommend" call — no implicit absence.
- [ ] An override on the recommendation is not silently replaced by the next synthesis pass.
- [ ] The recommendation traces back to the urgency signals it weighed; the bird-dogger can inspect the basis.

## Dependencies

- J001-O004-R001 — R003 reads the ranking R001 produces.
- J001-O004-R004 — R003 consumes prior triage state when synthesizing the current recommendation (carry-forward feeds synthesis).
- J001-O003-R003 — status confidence is a load-bearing input to R003's synthesis; low confidence must not feed a confident intervention call.

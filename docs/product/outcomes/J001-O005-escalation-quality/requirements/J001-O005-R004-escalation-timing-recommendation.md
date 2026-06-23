# J001-O005-R004 - Escalation Timing Recommendation

The product must synthesize signals — slip status, intervention recommendation, response window — into a recommendation of when an item warrants escalation, which the bird-dogger can adopt or override, rather than leaving the timing call entirely to the bird-dogger each checkpoint.

## Detail

Without a synthesized timing call, the bird-dogger re-derives the escalate-or-not judgment from raw signals every checkpoint. That re-derivation produces both premature escalations (RSK002) and late ones (RSK003) — the same threshold problem O004-R003 mitigates for intervention.

R004 names the *what*: a per-item recommendation of "escalate now," "not yet," or "cannot recommend," derivable from slip signals (O001), the intervention recommendation (O004-R003), the response window (R003), and prior escalation history (R005). The bird-dogger can adopt or override it. The override is recommendation-level and survives re-synthesis on later checkpoints in the same shape O003-R004 and O004-R003 commit to.

R004 does not prescribe the synthesis rule (weighting between slip, intervention, and response signals; the timing threshold). The rule is engineering-layer; R004 commits to producing a recommendation that traces to the signals it weighed.

The three recommendation states are a closed set for the same reason O004-R003 holds a closed set: the override needs a stable target and list-level rendering must show a known state.

## Edge Cases

- Signals insufficient (item in O004-R003 "cannot recommend," or response window unknown per R003): surfaced as "cannot recommend" per [PRD001](../../../pdrs/PRD001-coverage-over-precision.md), not defaulted to "not yet."
- Bird-dogger has overridden the recommendation on a prior checkpoint and the next synthesis produces a different call: the override is not silently replaced; the new synthesis output is surfaced alongside the override for the bird-dogger to adopt or dismiss. Same shape as O004-R003.
- Owner contact is stale (R002 shows long-unconfirmed owner): this does not block a recommendation, but feeds the synthesis as reduced confidence; R004 surfaces "cannot recommend" if the staleness leaves no usable target signal.

## Examples

### Recommendation under an open response window

- Input: item with red slip signal; intervention recommended yesterday under O004-R003; response window 8 hours since the bird-dogger's last touch.
- Expected Output: R004 surfaces "not yet" with the open response window as a load-bearing reason — escalating before the window closes would burn credibility (RSK002). The bird-dogger can adopt or override.
- Verification: the checkpoint review on the item shows the recommendation state and the response window together, not a timing call independent of the window signal.

## Acceptance Criteria

- [ ] Each item carries an explicit "escalate now," "not yet," or "cannot recommend" call — no implicit absence.
- [ ] An override on the recommendation is not silently replaced by the next synthesis pass.
- [ ] The recommendation traces back to the signals it weighed; the bird-dogger can inspect the basis.

## Dependencies

- J001-O005-R001 — without a target, an "escalate now" call has nowhere to land; R004 reads the chain when synthesizing.
- J001-O005-R002 — owner-confirmation staleness feeds synthesis as reduced target confidence; R004 surfaces "cannot recommend" when staleness leaves no usable target signal.
- J001-O005-R003 — response window is a load-bearing input; escalating inside an open window is the RSK002 failure R004 exists to prevent.
- J001-O005-R005 — prior escalation history feeds synthesis; re-escalating a path that already failed is the RSK004 failure to avoid.
- J001-O004-R003 — intervention recommendation is the upstream call; escalation is the next-tier action when intervention is already established as warranted.
- J001-O001-R001 — slip status feeds the timing call; recovery windows close as slip deepens.

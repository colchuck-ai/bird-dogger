# J001-O004-R001 - Urgency Ranking

The product must rank open items by urgency of intervention, with enough granularity to differentiate among items rather than only flagging "needs attention" vs "ok."

## Detail

R001 is the ordering commitment. Items in the active set are placed on a granularity that lets the bird-dogger pick the next target without inspecting every flagged one — strictly more than two buckets.

R001 names the *what*: ranked, with differentiating granularity. It does not prescribe the scale (categorical vs. numeric), the number of bands, or the weighting between signals. Those are held for the same reason O003-R003 confidence scale is held — the requirement layer commits to the shape, not the scale.

R001 is the ranking hub. The signals it consumes are surfaced under R002; the synthesis into an "intervene now" call is owned by R003; prior triage decisions that feed into ranking are carried by R004.

## Edge Cases

- Item lacks enough signal to be ranked at all: surfaced in a "cannot rank" state per [PRD001](../../../pdrs/PRD001-coverage-over-precision.md), not silently sorted to a default position at the bottom or top.
- Signals conflict (sources disagree on slip status per O003-R006, or trajectory contradicts status): the ranking reflects the conflict explicitly rather than picking the most confident signal silently. Same posture as PRD001.
- Two or more items tie on every signal: surfaced as equivalent rather than imposing a spurious ordering. The granularity commitment is "more than two buckets," not "no ties."
- Item is blocked by another open item (in-set or out-of-set per R005): the dependency is one of the inputs to urgency; an item whose only path forward is blocked does not silently outrank its blocker.

## Examples

### Checkpoint with mixed signal strength

- Input: 12 open items; 3 with recent slip and red status, 2 with no signal since registration, 7 with mixed mid-confidence signals.
- Expected Output: the 3 hot items rank highest with the 2 no-signal items in a distinct "cannot rank" state, not trailing at the bottom; the 7 mid-confidence items appear with differentiated positions, not all tied.
- Verification: the bird-dogger can pick the next item to triage without inspecting all 12.

## Acceptance Criteria

- [ ] Items appear in an order with more than two distinct levels of urgency.
- [ ] Items lacking enough signal to rank surface in a distinct "cannot rank" state, not at a default rank position.
- [ ] The ranking is reproducible from the same inputs — same signals produce the same order on a repeat assessment.

## Dependencies

- (none intra-outcome — R001 is the ranking hub other requirements attach to)

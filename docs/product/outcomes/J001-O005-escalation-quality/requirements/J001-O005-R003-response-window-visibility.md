# J001-O005-R003 - Response Window Visibility

The product must show how long the owner has had to respond since the bird-dogger's last touch on each item.

## Detail

Escalating before the owner has had a fair window to respond burns credibility for future escalations on the same item (RSK002). The bird-dogger needs to see the response window per item to recognize when a touch has had time to land and when it has not.

R003 names the *what*: per item, the elapsed time since the bird-dogger's last touch on that item, visible alongside the item. R003 does not prescribe what counts as a "touch" event in detail (comment, direct message, mention, meeting note), the fairness threshold, or whether the window is shown as elapsed time, a deadline, or both — those are engineering-layer. R003 reads "last touch" from the escalation record under R005; touches that are not escalation attempts (e.g., a routine nudge) are also in scope as anchors and are recorded the same way.

R003 is the response-side parallel of O003-R002 freshness — a per-item elapsed signal — but the anchor is the bird-dogger's last touch, not assessment or underlying-data change. R003 does not adopt the two-timestamp shape from [PRD002](../../../pdrs/PRD002-per-item-freshness-presentation.md); it carries one elapsed value.

## Edge Cases

- No recorded touch on the item yet: surfaced as "no touch recorded" per [PRD001](../../../pdrs/PRD001-coverage-over-precision.md), not defaulted to "fully responsive" or zero elapsed.
- Owner has responded since the last touch: response window resets or closes; the response itself is visible alongside the window (R005 carries the response record).
- Item has multiple recent touches (escalation chain followed): R003 carries the elapsed time since the most recent touch on the item; per-target windows live under R005's history view.

## Examples

### Mid-checkpoint review of a quiet item

- Input: item with a comment from the bird-dogger 3 days ago; no response from the owner since.
- Expected Output: list view shows "3 days since last touch, no response"; the bird-dogger can see at a glance whether the owner has had a fair window without opening the item.
- Verification: the bird-dogger can pick out items where the response window is still open vs. ones where it has clearly closed.

## Acceptance Criteria

- [ ] Each item shows the elapsed time since the bird-dogger's last touch, visible at the list level.
- [ ] Items with no recorded touch surface that gap explicitly rather than reading as zero elapsed.
- [ ] When a response has been received, the window is closed or reset and the response is visible alongside the item.

## Dependencies

- J001-O005-R005 — R003 reads "last touch" and "response received" from the escalation record R005 maintains.

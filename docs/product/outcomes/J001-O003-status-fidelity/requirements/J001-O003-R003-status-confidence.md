# J001-O003-R003 - Status Confidence

The product must surface a confidence level for any inferred
status.

## Detail

Inferred statuses are not equal. An inference from many recent,
agreeing signals is different from an inference from one stale
signal or from sources that partly contradict. Treating both as
"the status" hides that difference and lets weak inferences pass
as fact.

R003 names the *what*: each inferred status carries a confidence
level the bird-dogger can read alongside the status. The level
must differentiate among inferences, not collapse to a single
binary. R003 does not prescribe numeric vs. categorical, the
scale, or how the level is computed.

Confidence applies only to inferred status. An override (R004) is
the bird-dogger's own statement and carries no inferred
confidence.

## Edge Cases

- Status set by override (R004): no inferred confidence; the
  basis (R001) shows the override and confidence is either
  omitted or labelled "manual" — not silently set to "high."
- Sources disagree (R006): confidence reflects the conflict
  (low), not the average of the disagreeing readings.
- Single-signal inference: confidence reflects the thinness of
  the evidence, not defaulted to "high" because the one signal
  is recent.

## Examples

### Sparse-signal item

- Input: one PR comment 6 days ago; no commits; no other signal.
- Expected Output: inferred status "in-progress" surfaced at low
  confidence; bird-dogger can prioritize re-checking this item
  over items with high-confidence statuses.
- Verification: the confidence level is visible at the list
  level without opening the item.

## Acceptance Criteria

- [ ] Each inferred status carries a confidence level visible
      alongside the status.
- [ ] Confidence differentiates among levels rather than only
      flagging "confident" vs "not."
- [ ] Manually overridden statuses do not carry inferred
      confidence.

## Dependencies

- J001-O003-R001 — confidence attaches to the basis the status
  was inferred from.

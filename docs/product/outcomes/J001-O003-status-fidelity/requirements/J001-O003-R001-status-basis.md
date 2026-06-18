# J001-O003-R001 - Status Basis

The product must show the inputs and reasoning that produced each
item's tracked status.

## Detail

A status without a basis is opaque: the bird-dogger cannot tell
whether it is right, stale, sourced from the wrong place, or the
product of a rule that no longer fits the item.

R001 names the *what*: for each item, both the input signals (which
sources, which fields, at what time) and the reasoning that turned
them into a status are inspectable. It does not prescribe layout
(inline annotation vs. drill-down), how granular the reasoning
trace must be, or whether basis is shown at the list level or only
on the item view.

Basis includes overrides (R004) explicitly — an overridden status
must trace back to the override, not present as if inferred.

## Edge Cases

- Manually overridden status: basis shows the override as the
  current basis; the prior inferred basis is recoverable, not
  erased.
- Single-signal status: basis still shows that single signal
  explicitly, not implied by absence.
- Source expected but unread at time of assessment: basis names
  the missing source rather than implying full evidence (links to
  the per-item source-availability shape under O001-R006).

## Examples

### PR-tracked item flagged blocked

- Input: tracker shows ticket "in review"; PR has no commits for
  11 days; last PR comment 9 days ago; CI red since the 9-day
  mark.
- Expected Output: status "blocked"; basis lists the three signals
  with timestamps and names the rule ("review-stuck + stalled
  commits = blocked").
- Verification: the bird-dogger can name the signals and the
  reasoning without opening the tracker or the PR.

## Acceptance Criteria

- [ ] Each tracked item's status traces back to its source
      signals without the bird-dogger opening the source.
- [ ] The reasoning that produced the status is inspectable, not
      only the inputs.
- [ ] An overridden status is distinguishable from an inferred
      status in the basis.

## Dependencies

- (none intra-outcome — R001 is the basis hub other requirements
  attach to)

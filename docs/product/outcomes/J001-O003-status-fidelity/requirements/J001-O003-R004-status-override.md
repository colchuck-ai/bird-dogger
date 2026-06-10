# J001-O003-R004 - Status Override

The product must let the bird-dogger correct or override an
item's tracked status.

## Detail

Inference is not always right and trackers lag reality. When the
bird-dogger has direct knowledge — owner conversation, side-channel
update, observed shipped artifact — they must be able to set the
truth themselves rather than wait for the tracker or the inference
to catch up.

R004 names the *what*: the bird-dogger can set, change, or clear a
status override on any tracked item, and that override stands as
the item's status until the bird-dogger clears it. R004 does not
prescribe the override UI, what statuses are overridable, or the
lifetime of an override.

The override is first-class in the basis (R001 traces it) and in
the freshness story (R002 timestamps the override as the last
assessment). Re-assessment under R005 must not silently replace
an override. When the inferred reading from the current basis
disagrees with an active override, the disagreement is surfaced
alongside the override — this is R004's scope, not R006's
(which owns source-vs-source disagreement only).

## Edge Cases

- Re-assessment (R005) produces a different status than the
  override: the override is held as the displayed status; the
  new inferred reading is surfaced alongside the override for
  the bird-dogger to adopt or dismiss; the override is not
  silently replaced.
- Override cleared: status reverts to the current inferred
  reading; the prior override is recoverable in basis history,
  not erased.
- Item with no source backing (a manual item from O002-R002):
  override is the only status the item can carry; behaves as
  the normal override path, not a separate mode.

## Examples

### Tracker lag corrected by direct knowledge

- Input: tracker shows "open"; owner told the bird-dogger the
  item shipped this morning.
- Expected Output: bird-dogger sets override "closed"; status
  reflects "closed" at the next checkpoint even though the
  tracker has not caught up; basis (R001) shows override + the
  prior inferred basis still recoverable.
- Verification: the item's checkpoint status matches truth on
  the day the work shipped, not on the day the tracker is
  updated.

## Acceptance Criteria

- [ ] The bird-dogger can set or clear a status override on any
      tracked item.
- [ ] An override status is not silently replaced by
      re-assessment.
- [ ] A cleared override remains recoverable in the basis
      history.
- [ ] When the inferred reading from the current basis disagrees
      with an active override, the disagreement is surfaced
      alongside the override.

## Dependencies

- J001-O003-R001 — override appears in the basis trace.

# J001-O001-R003 - Expected Trajectory

The product must let the bird-dogger record an expected
trajectory — such as a due date or expected pace — for each
tracked item.

## Detail

Expected trajectory is the reference line against which observed
progress is measured (by R004). Without it, slip has nothing to
be measured against.

"Trajectory" is intentionally broader than a due date alone:
some items have a single deadline; some have a pace (e.g.
weekly progress); some have a milestone sequence. The
requirement is that *some* recordable expected shape exists per
item — not that every item uses the same shape.

## Edge Cases

- Item already has a due date recorded in the underlying source:
  the product should adopt that as the starting trajectory rather
  than force the bird-dogger to re-enter it.
- Item has no recordable expected pace and no due date (genuinely
  open-ended exploratory work): the product accepts "no
  trajectory" as a recorded state, not as a gap; R004 then has
  nothing to compare against and R005 (silence) carries the
  slip signal.
- Trajectory changes mid-flight (date moves, pace re-baselined):
  prior trajectory is recoverable so a re-baseline is not
  indistinguishable from on-track progress.

## Examples

### Recording a due date for a cross-team request

- Input: bird-dogger opens an item with no expected trajectory
  and records "expected complete by 2026-07-01."
- Expected Output: the item's trajectory is persisted and is
  available as input to R004's comparison from that point
  forward.
- Verification: the same item, viewed at the next checkpoint,
  shows the recorded trajectory and is now eligible for
  trajectory-based slip surfacing.

## Acceptance Criteria

- [ ] The bird-dogger can record at least one form of expected
      trajectory (due date or pace) for any tracked item.
- [ ] An expected trajectory persists across checkpoints and is
      not overwritten by source-side data refreshes.
- [ ] Recording "no expected trajectory" is a distinct state
      from "not yet recorded," so R001 can surface the latter
      as a coverage gap.

## Dependencies

- J001-O001-R004 — the recorded trajectory is the reference
  line R004 compares observed progress against.

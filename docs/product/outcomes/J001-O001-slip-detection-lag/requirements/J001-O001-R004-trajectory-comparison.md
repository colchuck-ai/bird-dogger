# J001-O001-R004 - Trajectory Comparison

The product must compare each item's observed progress against its recorded expected trajectory.

## Detail

This is the active half of the trajectory pair. R003 captures the expected line; R004 produces the on-track / slipping reading by measuring observed progress against it. The output of this comparison is one of the inputs R001 surfaces.

"Compare" names the *what*: a per-item judgement of how observed progress stands against expected. It does not prescribe the comparison method (threshold rule, model, heuristic).

## Edge Cases

- Item has no recorded trajectory (R003 "no trajectory" or "not yet recorded"): comparison is not run for this item; trajectory-based slip is not asserted, and the item falls to R005 (silence) or R001's "cannot assess" state.
- Item's observed progress is ahead of expected: comparison yields "on-track" (or "ahead"); not surfaced as slipping by R001 on this signal alone.
- Trajectory was just re-baselined (per R003): comparison runs against the new trajectory; prior comparisons remain in history for explainability.

## Examples

### Past-due against a recorded due date

- Input: item with recorded trajectory "due 2026-06-01"; observed progress shows item still open on 2026-06-08; comparison runs.
- Expected Output: comparison yields "slipping" (past due, not closed); R001 includes the item in the surfaced slipping set.
- Verification: the bird-dogger sees the item in the slipping set without having to open it and check the date manually.

## Acceptance Criteria

- [ ] Every tracked item with a recorded expected trajectory has a current comparison reading.
- [ ] The comparison reading is one of a defined, finite set of states (e.g. on-track / slipping / cannot assess), not free text.
- [ ] An item without a recorded trajectory does not produce a misleading on-track reading; it produces "cannot assess from trajectory."

## Dependencies

- J001-O001-R003 — provides the trajectory the comparison measures against.
- J001-O001-R001 — consumes the comparison reading as one input to the surfaced slipping set.

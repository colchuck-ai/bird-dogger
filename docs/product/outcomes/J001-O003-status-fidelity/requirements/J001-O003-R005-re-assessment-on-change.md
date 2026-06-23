# J001-O003-R005 - Re-Assessment On Change

The product must re-assess an item's status when its underlying data has changed since the last assessment.

## Detail

A stale assessment drifts from reality as the underlying data moves. The fix is to trigger a fresh assessment when the data moves, so the gap between assessment time and data time (shown by R002) does not grow into divergence.

R005 names the *what*: when an item's underlying data has changed since the last assessment, the item is reassessed before its status is next surfaced for a checkpoint. R005 does not prescribe detection mechanism (poll, push, hash compare), how often to check for change, or how to batch re-assessments — these are engineering-layer.

Re-assessment writes a new last-assessment timestamp (R002), produces a fresh basis (R001), may shift confidence (R003), and may newly surface or resolve a source conflict (R006).

## Edge Cases

- Change in a source that is currently unavailable: re-assessment is deferred; the item's assessment is surfaced as stale rather than silently held as fresh (links to O001-R006 source-availability surfacing).
- Override (R004) in effect: re-assessment still runs; the result is surfaced as a candidate alongside the override, not applied silently.
- High-frequency churn on one source (many small changes in quick succession): re-assessment does not have to run per event, only such that the assessment is not older than the latest change at the point of surfacing.

## Examples

### Comment added between checkpoints

- Input: item last assessed Monday 9am; tracker comment added Tuesday 2pm; checkpoint runs Wednesday 9am.
- Expected Output: at the Wednesday checkpoint, the status reflects the Tuesday comment; R002 shows the assessment time as no older than Tuesday 2pm.
- Verification: the assessment timestamp shown at the Wednesday checkpoint is later than the most recent underlying change.

## Acceptance Criteria

- [ ] An item with new underlying data since its last assessment is reassessed before its status is next surfaced.
- [ ] Re-assessment under an active override surfaces the new reading as a candidate without silently overwriting the override.
- [ ] An item whose source is unavailable when a change is detected has its stale-assessment state surfaced, not hidden.

## Dependencies

- J001-O003-R002 — re-assessment uses last-assessment time to decide what is stale.
- J001-O002-R006 — active-set refresh is the upstream signal that surfaces newly-changed source data into the assessment loop.

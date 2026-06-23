# J001-O001-R001 - Proactive Slip Surfacing

The product must surface items that appear to be slipping without requiring the bird-dogger to inspect each item.

## Detail

The product is responsible for the first move. Between checkpoints, it evaluates tracked items against the slip signals defined in R003/R004 (trajectory comparison) and R005 (silence as signal), and brings the items it judges to be slipping to the bird-dogger's attention as a distinct set — not buried among healthy items.

When signals disagree on a single item — e.g., trajectory says on-track but silence says slipping, or vice versa — R001 reconciles them into a single per-item reading rather than presenting raw signal conflict to the bird-dogger. The reconciliation rule is engineering-layer; the requirement is that one combined reading per item is surfaced, not multiple signal-specific ones.

"Surface" names the *what*: items in scope are presented as slipping. It does not prescribe channel, layout, or visual treatment.

## Edge Cases

- No slipping items: the surfaced set is empty and explicitly says so, rather than showing the full active set.
- Item has insufficient signal to judge (no recorded trajectory, no recent updates, source unread): surfaced under a separate "cannot assess" state — not silently dropped, not treated as healthy.
- Item previously surfaced as slipping is now on-track: removed from the slipping set; its prior surfacing is recoverable in history.

## Examples

### Checkpoint review with one slipping item

- Input: 12 active items; 1 is past its R003-recorded due date, 11 are on-track.
- Expected Output: the surfaced slipping set contains the 1 past-due item; the bird-dogger does not have to open the other 11 to know they're fine.
- Verification: count of items the bird-dogger must open to find the slipping one = 1, not 12.

## Acceptance Criteria

- [ ] Given a tracked item meets at least one defined slip signal, the product includes it in the surfaced slipping set without manual inspection.
- [ ] Given the active set contains both slipping and healthy items, the slipping items are distinguishable from healthy ones as a set, not only per-item.
- [ ] Given no items are slipping, the surfaced set is explicitly empty rather than indistinguishable from "not yet checked."

## Dependencies

- J001-O001-R004 — trajectory comparison is one of the inputs this requirement surfaces.
- J001-O001-R005 — silence is one of the inputs this requirement surfaces.
- J001-O001-R006 — source-unavailability feeds the "cannot assess" state and is surfaced alongside, so absence of signal is not read as on-track.

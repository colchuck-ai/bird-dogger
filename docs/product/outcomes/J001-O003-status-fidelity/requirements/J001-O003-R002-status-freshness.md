# J001-O003-R002 - Status Freshness

The product must show when each item's tracked status was last assessed and when its underlying data was last updated.

## Detail

Two timestamps, kept distinct: (a) when the product last assessed the item, (b) when the underlying data behind that assessment last changed. The gap between them is the staleness signal — assessment recent but data old means the assessment still holds; data new but assessment old means something has moved since we last looked and the status may no longer reflect reality.

R002 names the *what*: both timestamps are shown per item, not collapsed into one "updated" field. It does not prescribe precision, display format, whether the gap itself is computed and surfaced separately, or — for items whose basis spans multiple sources with different update times — how the per-item "underlying data was last updated" value is picked from across sources. The picking rule (max-of, per-source, or composite) is engineering-layer; R002 commits only to surfacing a single per-item value.

R002 is the per-item parallel of O001-R002 (Assessment Freshness) applied to status rather than slip. Two outcomes now carry the "last assessed + last underlying change" presentation pattern. Flagged for PDR consolidation.

## Edge Cases

- Source does not expose a data-update time: the assessment timestamp is shown; the data-update timestamp is surfaced as "unknown," not omitted or set to assessment time.
- Item never assessed (just added): both timestamps shown explicitly as "not yet assessed," distinguishable from a rendering bug or empty field.
- Status set by override (R004): "last assessed" reflects the override time; the underlying data-update timestamp continues to track the source independently.

## Examples

### Checkpoint review of a quiet item

- Input: item assessed Monday 9am; underlying PR last commented Saturday 2pm.
- Expected Output: list view shows "assessed Mon 9am, data Sat 2pm"; bird-dogger can see the assessment is post-data and there is nothing newer to re-read.
- Verification: the bird-dogger can compare the two without opening the item.

## Acceptance Criteria

- [ ] Each item shows both its last-assessment time and its last-data-update time.
- [ ] Unknown data-update time is explicit, not blank or implied to match assessment time.
- [ ] Override timestamps update the assessment time without overwriting the data-update time.

## Dependencies

- (none intra-outcome — R005 consumes R002, not the reverse)
- J001-O001-R002 — parallel per-item freshness pattern under a different outcome; flagged for PDR consolidation.

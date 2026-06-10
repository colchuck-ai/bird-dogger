# J001-O001-R006 - Source Availability

The product must surface when a known data source for an item
is unavailable or unread.

## Detail

If a source the product expects to read is silent — outage,
auth failure, integration not yet built, rate limit — the
items in it cannot be assessed for slip. Without this
requirement, those items would silently appear on-track
because no slip evidence reached the product.

"Unavailable" and "unread" are both in scope: the source is
known but the product currently cannot or did not get its
data, for any reason.

## Edge Cases

- Source has never been read (newly registered, first sync
  pending): surfaced as unread, not unavailable.
- Source is partially available (some items read, others
  failed): per-item rather than source-wide; items that were
  not read show the unavailability on themselves.
- Source is permanently retired: still surfaced until the
  bird-dogger explicitly removes it from coverage, so that
  retirement is a deliberate act, not a silent gap.

## Examples

### Source outage at checkpoint

- Input: tracked source returns errors for the last 6 hours;
  all 4 items from it could not be assessed.
- Expected Output: the 4 items show source-unavailable; they
  are not in the on-track set; they are visible alongside the
  surfaced slipping set per R001.
- Verification: the bird-dogger sees the 4 unread items at the
  checkpoint and can decide whether to chase the source itself
  or the items by another channel.

## Acceptance Criteria

- [ ] Given a known source is unavailable, items dependent on
      that source are not shown as on-track.
- [ ] Source-unavailable items are distinguishable from both
      on-track and slipping items at the list level.
- [ ] An unread source surfaces as unread without requiring the
      bird-dogger to open each item from it.

## Dependencies

- J001-O001-R001 — surfaced unavailability appears alongside
  the slipping set so absence of signal is not read as healthy.
- J001-O001-R002 — freshness reflects last successful
  assessment; an unavailable source freezes freshness rather
  than advancing it.
- J001-O001-R007 — sources become known by registration.

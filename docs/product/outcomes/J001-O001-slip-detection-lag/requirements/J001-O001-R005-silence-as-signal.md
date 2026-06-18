# J001-O001-R005 - Silence As Signal

The product must treat sustained absence of updates on an open
item as a slip indicator, not as steady progress.

## Detail

Silence here means: no new activity on an item over a window
the product judges meaningful. The default reading of "no
news" must be "potential slip," not "fine." This is the
mitigation for owner silence — an item without updates may
already be slipping, but absence of new information is
otherwise indistinguishable from steady progress.

"Sustained absence" names the *what*; the specific window and
its inputs (item type, expected pace, prior cadence) are an
engineering-layer concern.

## Edge Cases

- Item is closed: silence on a closed item is not a slip
  signal; it's the expected post-resolution state.
- Item is brand new (just added, no prior activity baseline):
  the silence window has no prior cadence to measure against;
  the product treats this as "cannot assess from silence" until
  enough history exists, rather than instantly flagging slip.
- Item has a recorded trajectory (R003) that explicitly allows a
  long quiet period (e.g. waiting on a fixed external date):
  the silence reading is reconciled with trajectory; R001
  resolves the combined reading.

## Examples

### Quiet item flagged at checkpoint

- Input: open item, last activity 21 days ago, prior typical
  cadence ~weekly, no recorded trajectory contradicting that
  pace.
- Expected Output: the item is treated as slipping on the
  silence signal; R001 surfaces it.
- Verification: the same item, with activity 2 days ago, would
  not be flagged on this signal alone.

## Acceptance Criteria

- [ ] Given an open item with no recent activity beyond the
      product's silence window, the item is treated as
      potentially slipping, not on-track.
- [ ] The silence reading does not require the bird-dogger to
      open each item to apply it.
- [ ] Closed items do not produce silence-based slip readings.

## Dependencies

- J001-O001-R001 — consumes the silence reading as one input to
  the surfaced slipping set.

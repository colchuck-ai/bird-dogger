# J001-O005-R002 - Owner Currency

The product must show how recently each item's recorded owner was
confirmed.

## Detail

A recorded owner who has rotated off the work, changed teams, or
gone on leave is the misdirected-target failure (RSK001). The fix
is to make the staleness of the owner record visible, so escalation
decisions account for whether the owner is still the right contact.

R002 names the *what*: per item, a timestamp (or equivalent
freshness signal) showing when the recorded owner was last
confirmed. R002 does not prescribe how confirmation happens
(bird-dogger acknowledgment, source re-read, owner reply observed),
how often confirmation must occur, or the staleness threshold —
those are engineering-layer.

R002 is the owner-side cousin of O003-R002 (status freshness): a
"how recently did we look at this signal" presentation, scoped to
owner rather than status. The two-timestamp shape from
[PRD002](../../../pdrs/PRD002-per-item-freshness-presentation.md)
does not transfer directly — there is no "underlying owner-change
time" most sources expose — so R002 commits to confirmation time
only.

## Edge Cases

- Owner never confirmed (just recorded from a source at
  registration): surfaced as "owner not yet confirmed" per
  [PRD001](../../../pdrs/PRD001-coverage-over-precision.md), not
  treated as confirmed at registration time.
- Owner field changed in the source between checkpoints:
  confirmation time resets; the change itself is visible alongside
  the freshness signal rather than silently swapping the recorded
  owner.
- No owner recorded (per R001): R002 has nothing to time; the "no
  owner known" state from R001 carries.

## Examples

### Stale-owner item in a long-running effort

- Input: item registered 6 months ago; owner field last confirmed
  4 months ago; no recent confirmation event.
- Expected Output: list view shows owner + a "confirmed 4 months
  ago" freshness signal; the bird-dogger can see the staleness
  without inspecting the source.
- Verification: the bird-dogger can tell at a glance which items
  carry a fresh owner contact and which need re-confirmation
  before escalation.

## Acceptance Criteria

- [ ] Each item with a recorded owner shows when that owner was
      last confirmed.
- [ ] Items with an owner that has never been confirmed surface
      that explicitly, not as confirmed-at-registration.
- [ ] The owner-confirmation signal is visible at the list level,
      not only on the item detail view.

## Dependencies

- J001-O005-R001 — R002 times the owner R001 records; with no
  owner, R002 has nothing to time.

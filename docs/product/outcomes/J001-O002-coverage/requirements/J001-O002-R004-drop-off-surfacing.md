# J001-O002-R004 - Drop-Off Surfacing

The product must surface items that have left the active set since
the last checkpoint and let the bird-dogger keep them in coverage.

## Detail

Items leave the active set for many reasons — a tracker auto-closes
them, an owner marks them resolved, a filter ages them out, an
archive sweep moves them. Some of those exits reflect real
completion; others are administrative and the work is still open.
If exits are silent, real work falls out of coverage.

R004 names the *what*: between checkpoint N and checkpoint N+1,
items that were in the active set at N but not at N+1 are surfaced
as a distinct set, with the reason they left when known. The
bird-dogger can keep any of them in coverage — either by reverting
the exit (where the source allows) or by promoting the item to a
manually captured entry (R002).

## Edge Cases

- Item exited because it was legitimately closed and verified:
  surfaced once, with the close reason; the bird-dogger can
  confirm and the product stops surfacing it.
- Item exited because a tracker auto-archived it but work is
  ongoing: surfaced; bird-dogger can return it to coverage via
  manual capture or, if the source supports it, by un-archiving.
- Many items dropped off in one checkpoint (e.g., a bulk
  tracker rule fired): the set is still surfaced as a set, not
  paginated into invisibility, and bulk actions are available.

## Examples

### Auto-archive sweep mid-quarter

- Input: a watched tracker archives 6 items overnight because of
  an inactivity rule; 2 of them still have open work.
- Expected Output: at the next checkpoint, all 6 appear as
  drop-offs with the archive reason; the bird-dogger keeps 2 in
  coverage and confirms 4 as legitimately exited.
- Verification: the 2 kept items remain in the active set; the
  4 confirmed exits no longer surface as drop-offs.

## Acceptance Criteria

- [ ] Items that left the active set between two checkpoints are
      surfaced as a distinct drop-off set at the next checkpoint.
- [ ] The reason for the exit is shown when the source provides
      one.
- [ ] The bird-dogger can keep a dropped-off item in coverage in
      one action, without losing its history.
- [ ] Confirmed legitimate exits stop appearing as drop-offs
      after the bird-dogger acknowledges them.

## Dependencies

- J001-O002-R001 — drop-off is defined as exit from the active
  set shown by R001.
- J001-O002-R002 — kept drop-offs are promoted via manual
  capture when the source no longer holds them.

# J001-O002-R005 - Item Migration Linking

The product must follow an item when it moves between sources,
projects, or trackers, so the active set tracks the live item
rather than its original record.

## Detail

Items move: a ticket gets re-filed under a new project, a doc
action item turns into a tracker ticket, a tracker is migrated
wholesale to a different system. If the product keeps watching
the original record, the live work disappears from the active
set silently — the original goes quiet (and looks closed), while
the real item is open somewhere else.

R005 names the *what*: when a tracked item moves, the product
links the new record to the same active-set entry so coverage
continues without a gap. The link can be detected automatically
(via source-provided migration metadata) or proposed by the
product and confirmed by the bird-dogger (when the signal is
ambiguous).

## Edge Cases

- Ambiguous migration — the original record is closed and two
  plausible successor items exist: surfaced as a confirm-link
  decision; the product does not auto-pick.
- Migration to a source the product does not watch or support:
  surfaced as a candidate source via R003 so coverage can be
  extended; the active-set entry is held in a "follow pending"
  state until linked or released.
- Original record reopens after migration: surfaced as a
  conflict; the bird-dogger decides which record is live.

## Examples

### Re-filing across projects

- Input: a watched ticket is moved from project A to project B
  in the same tracker; the source records the move.
- Expected Output: the active-set entry now points at the new
  ticket; history of the original is preserved in the entry.
- Verification: at the next checkpoint, the item appears once,
  under its new location, with continuity of triage and slip
  history from before the move.

## Acceptance Criteria

- [ ] When a watched source reports an item migration, the
      active-set entry follows the new record without manual
      re-capture.
- [ ] When migration is ambiguous, the product surfaces a
      confirm-link decision rather than picking silently.
- [ ] Item history (slip assessments, triage state, escalation
      record) is preserved across a linked migration.

## Dependencies

- J001-O002-R001 — the active-set view reflects the linked
  successor, not the abandoned original.
- J001-O002-R003 — migrations into unwatched sources flow into
  gap discovery.

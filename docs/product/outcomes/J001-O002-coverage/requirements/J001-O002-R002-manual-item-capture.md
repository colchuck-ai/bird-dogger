# J001-O002-R002 - Manual Item Capture

The product must let the bird-dogger add an item to the active set
without requiring it to exist in any watched source.

## Detail

Commitments made in meetings, chat, or hallway conversations
often never reach a tracker. If coverage depends on a source
recording the item first, those commitments are invisible to
every checkpoint. R002 closes that gap: the bird-dogger can put
an item directly into the active set as soon as the commitment
is made.

A manually captured item is a first-class member of the active
set — eligible for slip assessment (O001), status tracking
(O003), triage (O004), and escalation (O005) — even if no
external source ever backs it.

## Edge Cases

- Same item captured twice manually: deduplicated by the
  bird-dogger's choice, not silently collapsed; the second
  capture surfaces the existing entry.
- Manual item later appears in a watched source: no automatic
  linking occurs; the bird-dogger must decide manually whether
  to retain the manual entry, remove it, or replace it with
  the source-backed record.
- Manual item with no owner recorded: still captured; downstream
  outcomes (e.g., O005 escalation target) flag the missing
  owner rather than block capture.

## Examples

### Capture from a meeting

- Input: in a meeting, a team lead commits to delivering a
  config change by Friday; nothing is filed in a tracker.
- Expected Output: the bird-dogger creates an item in the active
  set with title, owner, and expected trajectory; it appears at
  the next checkpoint.
- Verification: the item is in the active set view (per R001)
  without any source being registered for it.

## Acceptance Criteria

- [ ] The bird-dogger can add an item to the active set with no
      watched source backing it.
- [ ] A manually captured item participates in every active-set
      operation (assessment, triage, escalation) on equal
      footing with source-backed items.
- [ ] Capturing the same item twice does not silently create two
      entries; the bird-dogger is shown the existing one.

## Dependencies

- J001-O001-R003 — manual items still need an expected
  trajectory to be assessable for slip.

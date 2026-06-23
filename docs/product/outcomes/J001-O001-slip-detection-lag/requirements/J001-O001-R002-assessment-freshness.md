# J001-O001-R002 - Assessment Freshness

The product must show how recently each item's slip status was assessed.

## Detail

Assessment freshness is a per-item timestamp answering "when did the product last evaluate whether this item is slipping?" Distinct from O003-R002, which carries two other per-item timestamps: (a) when the item's tracked status was last assessed — same shape as this freshness, different object (item status, not slip status); (b) when the item's underlying source data was last updated — different object entirely (source write time). All three per-item timestamps coexist; this R002 owns the slip-status assessment time.

Freshness is shown wherever an item's slip state is shown — both in the surfaced slipping set (R001) and at the item level — so a "looks fine" reading cannot be silently based on a stale assessment.

## Edge Cases

- Item never assessed (newly tracked, never evaluated): freshness reads "not yet assessed," not blank and not "now."
- Assessment failed (source error, evaluation crashed): freshness reflects the last *successful* assessment; the failure is surfaced through R006.
- Multiple assessments since last view: freshness reflects the most recent one; intermediate assessments are not required to be exposed at this layer.

## Examples

### Stale assessment caught at checkpoint

- Input: item shows "on-track"; last assessed 9 days ago; checkpoints run weekly.
- Expected Output: the freshness reading is visible alongside the on-track status, so the bird-dogger sees the assessment predates the last checkpoint.
- Verification: the bird-dogger can tell, without opening the item, that the "on-track" reading is older than expected.

## Acceptance Criteria

- [ ] Every item with a tracked slip status also shows when that status was assessed.
- [ ] The freshness value updates when (and only when) a new assessment completes; it does not advance on view, refresh, or unrelated edits.
- [ ] An item never assessed shows a distinct "not yet assessed" state rather than a missing or zero timestamp.

## Dependencies

- J001-O001-R001 — the surfaced slipping set displays freshness alongside each item.
- J001-O001-R006 — source unavailability is the reason an assessment can fail; freshness reflects last success.

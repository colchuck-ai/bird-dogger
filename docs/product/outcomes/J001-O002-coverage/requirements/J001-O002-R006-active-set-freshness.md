# J001-O002-R006 - Active Set Freshness

The product must refresh the active set from watched sources before each checkpoint and show when the last refresh occurred.

## Detail

Between checkpoints, new items get created in watched sources. If the active set is not refreshed before the next checkpoint, those items never enter coverage and the checkpoint runs against a stale picture.

R006 names the *what*: a refresh happens before each checkpoint and pulls new items from every watched source into the active set; the time of the last refresh is shown alongside the active set so the bird-dogger can judge how current it is. It does not prescribe a polling interval or push vs. pull mechanism.

When a refresh cannot reach a source (outage, credential failure), that condition is surfaced per J001-O001-R006 (Source Availability); coverage is honest about what was — and was not — refreshed.

## Edge Cases

- A watched source is unreachable at refresh time: the active set reflects what was reachable; the unreachable source is flagged, and items from prior refreshes for that source are retained but marked stale.
- A refresh completes between checkpoints (background): the shown last-refresh time updates; the bird-dogger does not need to trigger anything manually.
- A new source is registered after the most recent scheduled refresh: an incremental refresh runs against that source so its items appear in the active set at the next checkpoint.

## Examples

### Checkpoint after a missed refresh

- Input: nightly refresh failed because a tracker was down; 4 new items were created in that tracker overnight.
- Expected Output: active-set view shows the last successful refresh time and flags the unreached source; the 4 new items are absent but the gap is visible, not silent.
- Verification: the bird-dogger can tell from the view alone that coverage is incomplete and why.

## Acceptance Criteria

- [ ] The active set is refreshed from watched sources before each checkpoint.
- [ ] The time of the last successful refresh is shown with the active set.
- [ ] Sources unreachable at refresh time are surfaced; items from those sources are not silently lost from the active set.

## Dependencies

- J001-O002-R001 — the active set this requirement refreshes is the one R001 makes visible.
- J001-O002-R007 — newly registered sources participate in refresh.
- J001-O001-R006 — source-unavailability surfacing during a refresh is handled by the slip-detection requirement of the same name and not duplicated here.

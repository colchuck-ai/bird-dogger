# J001-O002-R001 - Coverage Set Visibility

The product must show which sources are currently being watched and which items are in the active checkpoint set.

## Detail

Coverage is the population the bird-dogger is responsible for at each checkpoint. If that population is invisible, the bird-dogger cannot tell whether a missing item is closed, unwatched, or forgotten — every checkpoint starts from doubt.

R001 names the *what*: both the watched sources and the active item set are inspectable as sets, not only as filters or side-effects of other views. It does not prescribe layout, depth of detail, or whether the two sets are shown together.

## Edge Cases

- No sources watched yet: the watched-sources view is explicitly empty rather than absent, so "I haven't set this up" is distinguishable from "the view is broken."
- Source is watched but currently contains no items: the source is still listed; it is not collapsed into the active item set.
- Item is in the active set but its source has since been unregistered: the item is still shown, flagged as source-detached, not silently dropped.

## Examples

### First checkpoint after registering two sources

- Input: two sources watched; source A has 5 items, source B has 3; no manual items.
- Expected Output: watched-sources view lists A and B; active set view lists 8 items with their source labels.
- Verification: the bird-dogger can name every watched source and every active item without opening an item detail view.

## Acceptance Criteria

- [ ] The bird-dogger can see the full list of watched sources at checkpoint time without inspecting individual items.
- [ ] The bird-dogger can see the full active item set at checkpoint time without first filtering.
- [ ] An empty watched-sources or active-set view is explicitly shown as empty, not omitted.

## Dependencies

- J001-O002-R006 — freshness annotates the active set view shown here.
- J001-O002-R007 — registration creates the entries this view lists.

# J001-O001-R007 - Source Registration

The product must let the bird-dogger register a new data source for tracking items.

## Detail

A source is registered when the bird-dogger tells the product "watch this for items." Without registration, the product cannot know to read a source, and slip in that source is invisible (the integration-gap half of source blind spot).

This requirement is identical in wording to J001-O002-R007; that duplication is intentional — both outcomes need the same capability, and a single source registration serves both. That this surfaces as one capability under two outcomes is a standing cross-outcome flag for the engineering layer, not a defect at the requirement layer.

## Edge Cases

- Source type the product does not yet support: registration records the intent and surfaces the gap; the product does not silently accept and then never read.
- Duplicate registration of the same source: deduplicated; the source is watched once, not multiple times.
- Registration succeeds but credentials/permissions are insufficient: registered, but reads fail; R006 surfaces the unavailability per item.

## Examples

### Adding a new tracker mid-quarter

- Input: bird-dogger registers a new tracker URL that holds three open items.
- Expected Output: the source is added to the watched set; on the next read cycle, its items are eligible for assessment and inclusion in R001's surfaced set.
- Verification: at the next checkpoint, items from the new source appear in the active view with assessable slip status.

## Acceptance Criteria

- [ ] The bird-dogger can register a new source without developer involvement for source types the product supports.
- [ ] A registered source enters the read cycle and its items become eligible for slip assessment.
- [ ] A registration request for an unsupported source type is explicitly surfaced, not silently dropped.

## Dependencies

- J001-O001-R006 — registered sources are the population over which availability is reported.

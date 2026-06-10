# J001-O002-R007 - Source Registration

The product must let the bird-dogger register a new data source
for tracking items.

## Detail

A source is registered when the bird-dogger tells the product
"watch this for items." Without registration, the product cannot
read a source and any item living there is outside coverage.

This requirement is identical in wording to J001-O001-R007;
that duplication is intentional — both outcomes need the same
capability, and a single source registration serves both. That
this surfaces as one capability under two outcomes is a standing
cross-outcome flag for the engineering layer, not a defect at
the requirement layer.

For coverage, the relevant consequence of registration is that
the source enters the refresh cycle (R006) and its items become
candidates for the active set (and surface in R001's
watched-sources view).

## Edge Cases

- Source type the product does not yet support: registration
  records the intent and surfaces the gap; the product does not
  silently accept and then never read.
- Duplicate registration of the same source: deduplicated; the
  source is watched once, not multiple times.
- Registration succeeds but credentials/permissions are
  insufficient: registered, but reads fail; the unavailability
  surfaces per R006's freshness signal and per J001-O001-R006.

## Examples

### Adding a new tracker mid-quarter

- Input: bird-dogger registers a new tracker URL that holds
  three open items.
- Expected Output: the source appears in the watched-sources
  view (R001); at the next refresh (R006) its items appear in
  the active set.
- Verification: at the next checkpoint, items from the new
  source are part of the active set.

## Acceptance Criteria

- [ ] The bird-dogger can register a new source without
      developer involvement for source types the product
      supports.
- [ ] A registered source enters the refresh cycle and its
      items become eligible for the active set.
- [ ] A registration request for an unsupported source type is
      explicitly surfaced, not silently dropped.

## Dependencies

- J001-O001-R007 — identical capability under the slip-detection
  outcome; one implementation serves both.
- J001-O002-R001 — registered sources appear in the
  watched-sources view.
- J001-O002-R006 — registered sources enter the refresh cycle.
- J001-O002-R003 — gap discovery is the primary feeder of
  registration requests beyond manual entry.

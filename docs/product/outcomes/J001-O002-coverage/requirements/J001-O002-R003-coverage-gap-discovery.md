# J001-O002-R003 - Coverage Gap Discovery

The product must surface candidate sources — sources referenced by
watched items or otherwise reachable from watched data but not
themselves watched — and candidate items — items present in
watched sources but excluded from the active set — so the
bird-dogger can decide whether to add them.

## Detail

Coverage gaps come in two shapes: a *source* the product can see
referenced (a linked tracker, a project mentioned on a watched
item, an upstream board) but is not watching, and an *item* the
product is already reading from a watched source but has not
brought into the active set (filtered out, newly created,
previously dismissed).

R003 names the *what*: both kinds of candidates are surfaced as
decisions for the bird-dogger to accept or reject — not silently
added (which would explode coverage) and not silently ignored
(which would leave gaps).

The bird-dogger's accept of a candidate source flows into R007
(Source Registration); accept of a candidate item flows into the
active set.

## Edge Cases

- Candidate source of an unsupported type: surfaced as a
  candidate with the gap noted, so the bird-dogger knows the
  product cannot watch it yet, rather than the candidate being
  hidden.
- Candidate item is a duplicate of a manually captured item
  (R002): surfaced as a link/merge candidate, not a new entry.
- Candidate was previously rejected by the bird-dogger: the
  product remembers the rejection and does not re-surface it at
  every checkpoint unless something material about the candidate
  has changed.

## Examples

### Linked tracker discovered from a watched item

- Input: watched item references a related ticket in a tracker
  the bird-dogger does not yet watch.
- Expected Output: the referenced tracker appears as a candidate
  source with the trigger (which watched item referenced it)
  shown; accepting it registers the source per R007.
- Verification: the candidate appears once accepting it adds the
  tracker to the watched-sources view per R001.

## Acceptance Criteria

- [ ] The product surfaces candidate sources reachable from
      watched data, with the trigger that made them candidates.
- [ ] The product surfaces candidate items present in watched
      sources but not in the active set.
- [ ] Accepting a candidate source results in it being watched
      per R007; accepting a candidate item results in it being in
      the active set.
- [ ] Rejected candidates are remembered and not re-surfaced at
      every checkpoint without material change.

## Dependencies

- J001-O002-R001 — gap discovery is defined relative to the
  watched/active sets shown by R001.
- J001-O002-R007 — accepted candidate sources are registered via
  R007.
- J001-O002-R002 — candidate items may collide with manually
  captured items and need link/merge handling.

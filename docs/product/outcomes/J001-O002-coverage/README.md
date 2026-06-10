# J001-O002 - Coverage

Minimize the likelihood that an open item is forgotten between
checkpoints.

## Risks

- **J001-O002-RSK001** - Implicit Commitment: A commitment is made in
  conversation, meeting, document, or chat but is never recorded as a
  trackable item, so it does not exist in any list the bird-dogger can
  check, increasing the likelihood that the item is forgotten between
  checkpoints.
- **J001-O002-RSK002** - Unwatched Source: An item lives in a data
  source that has not been registered for the bird-dogger's coverage,
  so items in that source are outside every checkpoint, increasing the
  likelihood that the item is forgotten between checkpoints.
- **J001-O002-RSK003** - List Drop-Off: An item leaves the active
  checkpoint view — closed, archived, filtered, or aged out by tracker
  rules — while real work on the item remains, so the next checkpoint
  does not include it, increasing the likelihood that the item is
  forgotten between checkpoints.
- **J001-O002-RSK004** - Item Migration Loss: An item moves between
  sources, projects, or trackers and the bird-dogger continues to
  watch the old location, so the live item is not in the active view,
  increasing the likelihood that the item is forgotten between
  checkpoints.
- **J001-O002-RSK005** - Active Set Drift: The active checkpoint set
  is not refreshed against watched sources between checkpoints, so
  newly created items in those sources are missing from the next
  checkpoint's list, increasing the likelihood that the item is
  forgotten between checkpoints.

## Requirements

- **J001-O002-R001** - Coverage Set Visibility: The product must show
  which sources are currently being watched and which items are in
  the active checkpoint set.
- **J001-O002-R002** - Manual Item Capture: The product must let the
  bird-dogger add an item to the active set without requiring it to
  exist in any watched source.
- **J001-O002-R003** - Coverage Gap Discovery: The product must
  surface candidate sources — sources referenced by watched items or
  otherwise reachable from watched data but not themselves watched —
  and candidate items — items present in watched sources but excluded
  from the active set — so the bird-dogger can decide whether to add
  them.
- **J001-O002-R004** - Drop-Off Surfacing: The product must surface
  items that have left the active set since the last checkpoint and
  let the bird-dogger keep them in coverage.
- **J001-O002-R005** - Item Migration Linking: The product must
  follow an item when it moves between sources, projects, or
  trackers, so the active set tracks the live item rather than its
  original record.
- **J001-O002-R006** - Active Set Freshness: The product must refresh
  the active set from watched sources before each checkpoint and
  show when the last refresh occurred.
- **J001-O002-R007** - Source Registration: The product must let the
  bird-dogger register a new data source for tracking items.

## Risk-Requirement Map

- **J001-O002-RSK001**: J001-O002-R002
- **J001-O002-RSK002**: J001-O002-R001, J001-O002-R003, J001-O002-R007
- **J001-O002-RSK003**: J001-O002-R004
- **J001-O002-RSK004**: J001-O002-R005
- **J001-O002-RSK005**: J001-O002-R006

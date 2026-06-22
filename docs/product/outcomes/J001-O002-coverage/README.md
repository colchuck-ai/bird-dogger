# J001-O002 - Coverage

Minimize the likelihood that an open item is forgotten between
checkpoints.

## Coverage Scope

Coverage applies to items actively present in registered watched
sources at refresh time. Items that leave watched sources, are
archived by trackers, or migrate to unregistered sources exit the
active set by design; recovering them requires manual capture (R002)
or re-registration of the new source (R007).

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

## Accepted Risks

- **J001-O002-RSK002** (partial): Automated gap discovery — candidate
  sources and items outside the registered set are not surfaced
  automatically. The bird-dogger must identify and register new sources
  manually (R007). RSK002 is partially mitigated: visibility (R001) and
  registration (R007) are addressed; proactive gap discovery is out of
  scope.
- **J001-O002-RSK003**: List drop-off is accepted per the coverage scope
  rule. Items that leave watched sources are not tracked back; the
  active set reflects current contents of registered sources only. The
  bird-dogger can re-add dropped items manually (R002).
- **J001-O002-RSK004**: Item migration loss is accepted per the coverage
  scope rule. When an item moves to an unregistered source, it exits the
  active set; the bird-dogger must register the new source (R007) or
  recapture the item manually (R002).

## Requirements

- **J001-O002-R001** - Coverage Set Visibility: The product must show
  which sources are currently being watched and which items are in
  the active checkpoint set.
- **J001-O002-R002** - Manual Item Capture: The product must let the
  bird-dogger add an item to the active set without requiring it to
  exist in any watched source.
- **J001-O002-R006** - Active Set Freshness: The product must refresh
  the active set from watched sources before each checkpoint and
  show when the last refresh occurred.
- **J001-O002-R007** - Source Registration: The product must let the
  bird-dogger register a new data source for tracking items.

## Risk-Requirement Map

- **J001-O002-RSK001**: J001-O002-R002
- **J001-O002-RSK002**: J001-O002-R001, J001-O002-R007
- **J001-O002-RSK003**: *(accepted — see Accepted Risks)*
- **J001-O002-RSK004**: *(accepted — see Accepted Risks)*
- **J001-O002-RSK005**: J001-O002-R006

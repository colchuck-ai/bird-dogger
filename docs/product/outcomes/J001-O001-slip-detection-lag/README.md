# J001-O001 - Slip Detection Lag

Minimize the time between an item starting to slip and the bird-dogger noticing.

## Risks

- **J001-O001-RSK001** - Coarse Check Cadence: Items are observed at intervals; slip that begins between observations is not visible until the next one, increasing the time between an item slipping and the bird-dogger noticing.
- **J001-O001-RSK002** - Buried Slip Signal: Evidence that an item is slipping is present in tracked data but mixed in with healthy items, requiring the bird-dogger to search before recognizing the slip, increasing the time between an item slipping and the bird-dogger noticing.
- **J001-O001-RSK003** - No Expected Trajectory: Without a recorded expected pace or due date for an item, observed progress cannot be measured as on-track or slipping, increasing the time between an item slipping and the bird-dogger noticing.
- **J001-O001-RSK004** - Owner Silence: An item without recent updates may already be slipping, but the absence of new information is indistinguishable from steady progress, increasing the time between an item slipping and the bird-dogger noticing.
- **J001-O001-RSK005** - Source Blind Spot: Slip occurs in a data source the product cannot read — because of an outage or because the source has not been integrated — leaving the slip invisible, increasing the time between an item slipping and the bird-dogger noticing.

## Requirements

- **J001-O001-R001** - Proactive Slip Surfacing: The product must surface items that appear to be slipping without requiring the bird-dogger to inspect each item.
- **J001-O001-R002** - Assessment Freshness: The product must show how recently each item's slip status was assessed.
- **J001-O001-R003** - Expected Trajectory: The product must let the bird-dogger record an expected trajectory — such as a due date or expected pace — for each tracked item.
- **J001-O001-R004** - Trajectory Comparison: The product must compare each item's observed progress against its recorded expected trajectory.
- **J001-O001-R005** - Silence As Signal: The product must treat sustained absence of updates on an open item as a slip indicator, not as steady progress.
- **J001-O001-R006** - Source Availability: The product must surface when a known data source for an item is unavailable or unread.
- **J001-O001-R007** - Source Registration: The product must let the bird-dogger register a new data source for tracking items.

## Risk-Requirement Map

- **J001-O001-RSK001**: J001-O001-R001, J001-O001-R002
- **J001-O001-RSK002**: J001-O001-R001
- **J001-O001-RSK003**: J001-O001-R003, J001-O001-R004
- **J001-O001-RSK004**: J001-O001-R001, J001-O001-R005
- **J001-O001-RSK005**: J001-O001-R006, J001-O001-R007

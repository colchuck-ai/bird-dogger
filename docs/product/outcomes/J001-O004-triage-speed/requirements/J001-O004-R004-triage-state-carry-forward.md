# J001-O004-R004 - Triage State Carry-Forward

The product must record each item's prior triage decision and
resurface it at the next checkpoint alongside what has changed
since.

## Detail

Without carry-forward, the bird-dogger re-derives urgency from
raw signals every checkpoint, even when nothing has changed
(RSK004). Memory between checkpoints is the mitigation.

R004 names the *what*: each item's prior triage decision is
recorded, and at the next checkpoint that decision is shown
alongside what has changed since it was made. Decision states
the README enumerates — "watching," "deferred," "already
intervened," "awaiting reply" — are illustrative; R004 does not
prescribe the closed set.

"What has changed" is anchored on the per-item freshness shape
per
[PRD002](../../../pdrs/PRD002-per-item-freshness-presentation.md):
the gap between the prior triage time and the item's current
assessment + underlying-data timestamps. R004 also records the
context at the time of the decision (rank, signals,
recommendation) so the change comparison has a real anchor.
That recorded context is a snapshot R004 captures from R001,
R002, and R003 at decision-time and stores; R004 does not
re-read those requirements live for prior decisions. The
dependency on R001/R002/R003 is shape-of, not live-on, which
keeps R004 a memory hub rather than a synthesis hub.

R004 is the memory hub. R003 reads R004 (today's synthesis
consumes prior decisions); R004 does not read R003 of the same
cycle.

## Edge Cases

- Nothing has changed since the prior triage (signals stable,
  no new underlying data, recommendation unchanged): the item
  resurfaces with its prior decision and an explicit "no
  change" signal, not silently held off the triage list.
- Prior triage decision is stale (a "watching" decision across
  multiple checkpoints with no review): the staleness surfaces
  explicitly per
  [PRD001](../../../pdrs/PRD001-coverage-over-precision.md), not
  silently held as fresh.

## Examples

### Quiet item between checkpoints

- Input: an item triaged "watching" last checkpoint; no commits,
  no status change, no new source data since.
- Expected Output: item resurfaces with the "watching" decision
  and a "no change since" anchor; the bird-dogger sees that
  triage is still applicable without re-deriving it from raw
  signals.
- Verification: the bird-dogger can confirm the prior decision
  without inspecting the underlying source.

## Acceptance Criteria

- [ ] Each item with a prior triage decision shows that
      decision at the next checkpoint.
- [ ] "What has changed since" is computed against the per-item
      freshness shape from PRD002 (assessment + underlying-data
      timestamps), not a single rolled-up timestamp.
- [ ] Decisions unreviewed across multiple checkpoints surface
      as stale per PRD001 rather than implicitly holding as
      current.

## Dependencies

- (none intra-outcome — R004 is the memory hub; R003 consumes
  it, not the reverse)
- J001-O002-R006 — active-set refresh provides the cadence
  against which "what has changed" is computed; R004 reads
  refresh-time alongside the prior triage time.
- J001-O003-R002 — per-item status freshness is one of the
  signals R004 surfaces as part of "what has changed since."

# J001-O004-R005 - Inter-Item Dependency Surfacing

The product must surface dependencies between open items at the list level so triage accounts for which items are blocked on others.

## Detail

Triaging an item that cannot move because it is waiting on another open item is wasted intervention (RSK005). The dependency must be visible alongside the items being triaged so the bird-dogger picks targets that can actually move.

R005 names the *what*: dependencies between items in the active set are visible at the list level. Direction is asymmetric — "A depends on B" means A is blocked by B; intervening on A is likely premature until B moves.

R005 does not prescribe how dependencies are captured (manual entry, inferred from tracker links, source-imported) — that is engineering-layer. It commits to surfacing them and to a few specific shapes around edge cases (cycles, out-of-set dependencies, stale dependencies) where silent handling would mislead triage.

## Edge Cases

- Circular dependencies (A depends on B, B depends on A, possibly via a longer chain): the cycle surfaces as a cycle per [PRD001](../../../pdrs/PRD001-coverage-over-precision.md), not silently broken or hidden.
- Dependency points to an item outside the active set: surfaced as an external dependency, distinguishable from an in-set dependency, so the bird-dogger knows whether the blocker is even within their coverage (O002).
- Upstream item resolved but the dependency was not closed (upstream done, downstream still flagged as blocked): staleness of the dependency surfaces explicitly per PRD001 / [PRD002](../../../pdrs/PRD002-per-item-freshness-presentation.md), not silently held as still-blocking.

## Examples

### Triage list with a dependency chain

- Input: item A "blocked, needs intervention" with a dependency on item B (also in the active set, currently "in progress"); item C blocked on an item D outside the active set.
- Expected Output: A's row shows the dependency on B inline; C's row shows the dependency on D and marks it as external; the bird-dogger picks B (the actual movable blocker) as the next intervention target, not A.
- Verification: the bird-dogger can identify the right target from the list view without opening either item.

## Acceptance Criteria

- [ ] Each item with a dependency on another shows that dependency at the list level.
- [ ] In-set dependencies are distinguishable from dependencies on items outside the active set.
- [ ] Dependency cycles surface as cycles rather than being silently broken to render the list.

## Dependencies

- J001-O004-R001 — dependencies inform the ranking; an item whose only path forward is blocked should not rank above its blocker silently.
- J001-O004-R002 — dependencies are one of the list-level signals R002 surfaces; R005 names the dependency-specific shape on top of R002's general commitment.
- J001-O002-R001 — the coverage set scope defines what "in-set" means for the in-set vs. out-of-set distinction R005 makes.
- J001-O002-R006 — active-set refresh keeps "in-set" current between checkpoints; the out-of-set distinction relies on a refreshed picture, not the prior checkpoint's snapshot.

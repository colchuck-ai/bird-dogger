# J001-O004 - Triage Speed

Minimize the time to identify which items most need intervention right
now.

## Risks

- **J001-O004-RSK001** - No Urgency Ranking: Open items are not ranked
  by urgency of intervention, or are ranked only by a binary "needs
  attention" indicator that does not differentiate among them,
  requiring the bird-dogger to evaluate every flagged item to choose
  what to act on first, increasing the time to identify which items
  most need intervention right now.
- **J001-O004-RSK002** - Buried Urgency Signals: Facts that drive each
  item's urgency — slip indicators, freshness, due-date proximity,
  blocker age — are not visible alongside the items being triaged, so
  the bird-dogger must inspect each item individually before
  recognizing what is urgent, increasing the time to identify which
  items most need intervention right now.
- **J001-O004-RSK003** - Undefined Intervention Threshold: Urgency
  signals are available but no committed threshold defines when they
  collectively mean "needs intervention now," requiring the
  bird-dogger to derive that threshold from scratch each checkpoint,
  increasing the time to identify which items most need intervention
  right now.
- **J001-O004-RSK004** - Triage Memory Loss: Prior triage decisions —
  watching, deferred, already intervened, awaiting reply — are not
  carried forward between checkpoints, so the bird-dogger re-derives
  each item's urgency from raw signals every time, even when nothing
  has changed since the last checkpoint, increasing the time to
  identify which items most need intervention right now.
- **J001-O004-RSK005** - Hidden Item Dependencies: Dependencies between
  open items — when intervening on one is blocked until another moves
  — are not visible alongside the items being triaged, so the
  bird-dogger may mis-triage an item that cannot be moved yet,
  increasing the time to identify which items most need intervention
  right now.

## Requirements

- **J001-O004-R001** - Urgency Ranking: The product must rank open
  items by urgency of intervention, with enough granularity to
  differentiate among items rather than only flagging "needs
  attention" vs "ok."
- **J001-O004-R002** - List-Level Urgency Signals: The product must
  show the facts that drive each item's urgency at the list level,
  not only on the item's detail view.
- **J001-O004-R003** - Intervention Recommendation: The product must
  synthesize the urgency signals into a recommendation of which items
  need intervention right now, which the bird-dogger can adopt or
  override, rather than leaving the synthesis entirely to the
  bird-dogger each checkpoint.
- **J001-O004-R004** - Triage State Carry-Forward: The product must
  record each item's prior triage decision and resurface it at the
  next checkpoint alongside what has changed since.
- **J001-O004-R005** - Inter-Item Dependency Surfacing: The product
  must surface dependencies between open items at the list level so
  triage accounts for which items are blocked on others.

## Risk-Requirement Map

- **J001-O004-RSK001**: J001-O004-R001, J001-O004-R003
- **J001-O004-RSK002**: J001-O004-R002
- **J001-O004-RSK003**: J001-O004-R003
- **J001-O004-RSK004**: J001-O004-R004
- **J001-O004-RSK005**: J001-O004-R005

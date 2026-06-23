# J001-O005-R001 - Escalation Target

The product must surface, for each item, the current owner and the escalation chain above them.

## Detail

An escalation that lands on someone who cannot move the item fails on contact (RSK001). The bird-dogger needs to see who owns the item and who sits above that owner before deciding where to push next.

R001 names the *what*: per item, the recorded owner and the chain above that owner are visible. R001 does not prescribe how the chain is sourced (manual entry, org integration, inheritance from the source), how deep it goes, or what role labels it carries — those are engineering-layer.

R001 is the chain hub for O005. R002 stamps the owner with a confirmation time; R004 reads the chain when synthesizing timing; R005 records which target on the chain each attempt landed on.

## Edge Cases

- No owner recorded for the item: surfaced as "no owner known" per [PRD001](../../../pdrs/PRD001-coverage-over-precision.md), not defaulted to the source's nominal lead or left blank.
- Owner recorded but no chain above them: surfaced as "no chain known above owner" per PRD001, distinguishable from "owner is top of chain."
- Owner appears in multiple sources with different chains (cross-team item, dual reporting): the disagreement is visible rather than silently picking one chain.

## Examples

### Item with partial escalation context

- Input: item with a recorded owner from the tracker but no chain above them registered.
- Expected Output: list view shows owner + an explicit "no chain known above owner" marker; the bird-dogger sees the gap without opening the item.
- Verification: the bird-dogger can tell at a glance which items have a usable chain and which need chain entry before escalation.

## Acceptance Criteria

- [ ] Each item shows its current recorded owner.
- [ ] Each item shows the escalation chain above that owner, or an explicit gap marker when none is recorded.
- [ ] Items with no recorded owner surface that gap explicitly rather than reading as blank or defaulted.

## Dependencies

- (none intra-outcome — R001 is the chain hub other O005 requirements attach to)

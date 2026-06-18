# J001-O003 - Status Fidelity

Maximize the confidence that each item's tracked status reflects its
true status.

## Risks

- **J001-O003-RSK001** - Bad Status Inference: Automated interpretation
  of tracker signals produces a status that disagrees with the item's
  true state, reducing confidence that the tracked status reflects
  reality.
- **J001-O003-RSK002** - Stale Status Assessment: An item's underlying
  data changes after its status was last assessed, so the prior
  assessment no longer matches reality, reducing confidence that the
  tracked status reflects truth.
- **J001-O003-RSK003** - Lagging Tracker Data: The tracker is not
  updated to match an item's actual progress, so even a correctly-read
  status diverges from the true state, reducing confidence that the
  tracked status reflects reality.
- **J001-O003-RSK004** - Conflicting Signals: Sources disagree on an
  item's status, leaving the bird-dogger unsure which to trust,
  reducing confidence that the tracked status reflects reality.

## Requirements

- **J001-O003-R001** - Status Basis: The product must show the inputs
  and reasoning that produced each item's tracked status.
- **J001-O003-R002** - Status Freshness: The product must show when
  each item's tracked status was last assessed and when its underlying
  data was last updated.
- **J001-O003-R003** - Status Confidence: The product must surface a
  confidence level for any inferred status.
- **J001-O003-R004** - Status Override: The product must let the
  bird-dogger correct or override an item's tracked status.
- **J001-O003-R005** - Re-Assessment On Change: The product must
  re-assess an item's status when its underlying data has changed
  since the last assessment.
- **J001-O003-R006** - Conflict Surfacing: The product must flag when
  sources disagree on an item's status, rather than silently picking
  one.

## Risk-Requirement Map

- **J001-O003-RSK001**: J001-O003-R001, J001-O003-R003, J001-O003-R004
- **J001-O003-RSK002**: J001-O003-R002, J001-O003-R005
- **J001-O003-RSK003**: J001-O003-R001, J001-O003-R004
- **J001-O003-RSK004**: J001-O003-R001, J001-O003-R003, J001-O003-R006

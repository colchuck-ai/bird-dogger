# J001-O005 - Escalation Quality

Maximize the confidence that escalations land with the right person at
the right time.

## Risks

- **J001-O005-RSK001** - Misdirected Target: The escalation reaches
  someone who cannot move the item right now — because the recorded
  owner is no longer the right person, or because no escalation chain
  above the owner is recorded — reducing confidence that escalations
  land with the right person at the right time.
- **J001-O005-RSK002** - Premature Escalation: The escalation goes out
  before the owner has had a fair window to respond to the bird-dogger's
  last touch, eroding the credibility of future escalations on the
  same item, reducing confidence that escalations land with the right
  person at the right time.
- **J001-O005-RSK003** - Late Escalation: The escalation goes out after
  the window in which intervention could still recover the item has
  passed, reducing confidence that escalations land with the right
  person at the right time.
- **J001-O005-RSK004** - Escalation History Loss: Prior escalation
  attempts on an item — who was reached, when, through what channel,
  and how they responded — are not carried forward between checkpoints,
  so the bird-dogger may re-escalate a path that already failed or skip
  a step in the chain, reducing confidence that escalations land with
  the right person at the right time.

## Requirements

- **J001-O005-R001** - Escalation Target: The product must surface, for
  each item, the current owner and the escalation chain above them.
- **J001-O005-R002** - Owner Currency: The product must show how
  recently each item's recorded owner was confirmed.
- **J001-O005-R003** - Response Window Visibility: The product must
  show how long the owner has had to respond since the bird-dogger's
  last touch on each item.
- **J001-O005-R004** - Escalation Timing Recommendation: The product
  must synthesize signals — slip status, intervention window, response
  window — into a recommendation of when an item warrants escalation,
  which the bird-dogger can adopt or override, rather than leaving the
  timing call entirely to the bird-dogger each checkpoint.
- **J001-O005-R005** - Escalation Record: The product must record each
  escalation attempt — target, channel, time, and response — and
  surface that history at the item level.

## Risk-Requirement Map

- **J001-O005-RSK001**: J001-O005-R001, J001-O005-R002
- **J001-O005-RSK002**: J001-O005-R003, J001-O005-R004
- **J001-O005-RSK003**: J001-O005-R004
- **J001-O005-RSK004**: J001-O005-R005

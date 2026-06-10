# J001-O005-R006 - Escalation Contact Registration

The product must let the bird-dogger register an escalation contact for
an item.

## Detail

An item's escalation chain — owner and everyone above them — is only
useful if it can be populated. R001 surfaces what the product knows;
without a way to record what the bird-dogger knows, the chain stays
empty even when the bird-dogger has the information. That gap is the
chain-registration half of RSK001 (Misdirected Target): the risk
statement names two failure modes — recorded owner is no longer the
right person, or no escalation chain above the owner is recorded — and
R001 surfaces the second as "no chain known above owner" without giving
the bird-dogger any way to close it.

R006 names the *what*: the bird-dogger can record a person as an
escalation contact for a specific item, in a specific position relative
to the owner (the owner itself, or above the owner). R006 does not
prescribe how identities are sourced (manual entry, org integration,
free-form name + handle), how chain order is represented, what role
labels are carried, or how identity is reconciled across items — those
are engineering-layer.

R006 produces; R001 consumes. R006 is the capability-side mitigation of
RSK001; R001 (chain surfacing) and R002 (owner currency) are the
presentation-side mitigations. R006 does not stamp confirmation
freshness — that is R002's role for the owner. Chain-member staleness
above the owner is currently uncovered at the requirement layer; see
Edge Cases.

R006 is the same shape as O001-R007 / O002-R007 (source registration):
bird-dogger-driven registration of a data element the product cannot
infer on its own.
[PRD003](../../../pdrs/PRD003-shared-source-registration.md) sets the
pattern for shared-capability requirements — duplicate at the
requirement layer where two outcomes legitimately need it, share at the
component layer. R006 follows the shape but is single-outcome:
escalation contact registration is not currently needed under any
outcome but O005, so R006 lives only here. If a future outcome
legitimately needs the capability, PRD003's pattern applies.

## Edge Cases

- Bird-dogger registers an owner different from a source-derived owner:
  the bird-dogger's registration is recorded as the owner; the
  source-derived value is surfaced as a disagreement per
  [PRD001](../../../pdrs/PRD001-coverage-over-precision.md) — same shape
  as R001 EC3's multi-source chain disagreement — not silently dropped.
  R002 times the bird-dogger's confirmation of the registered owner.
- Duplicate registration of the same person on the same item's chain:
  deduplicated; the contact appears once in the chain, not multiple
  times.
- Contact registered without a role label or relationship description:
  registered; R001's role-label engineering-layer punt covers what the
  product does with that gap. Registration is not blocked on role-label
  completeness.
- Chain depth the product does not yet support: surfaced explicitly per
  [PRD001](../../../pdrs/PRD001-coverage-over-precision.md); the product
  does not silently truncate. Same posture as R007's "source type not
  supported."
- Contact registered, then leaves the organization or rotates off: out
  of R006's scope. R002 covers owner-confirmation staleness;
  chain-member staleness above the owner is currently uncovered at the
  requirement layer and is a standing engineering-layer concern.
- Registering a chain on an item that has no owner: allowed; R001
  continues to surface "no owner known" alongside the now-populated
  chain — populating the chain does not implicitly fill the owner slot.

## Examples

### Registering a chain for an item with a source-derived owner only

- Input: an item carries an owner from the source with no chain above
  them; the bird-dogger knows the owner reports to a director.
- Expected Output: the bird-dogger registers the director as the next
  contact above the owner; R001 surfaces the populated chain at the
  next checkpoint.
- Verification: at the next checkpoint, the item shows owner + director
  in the chain rather than "no chain known above owner," and R004's
  synthesis can read the director as an escalation target.

## Acceptance Criteria

- [ ] The bird-dogger can register an escalation contact for an item
      without developer involvement for chain shapes the product
      supports.
- [ ] A registered contact appears in R001's surfaced chain for the
      item.
- [ ] A registration request for a chain shape the product does not
      support is explicitly surfaced, not silently dropped.
- [ ] When the bird-dogger registers an owner over a source-derived
      one, both are visible; the source-derived value is not silently
      dropped.

## Dependencies

- J001-O005-R001 — R006 populates the chain R001 surfaces; the
  registered contact is meaningful only because R001 presents it.

# J001-O003-R006 - Conflict Surfacing

The product must flag when sources disagree on an item's status, rather than silently picking one.

## Detail

Multi-source items can carry contradictory signals — the tracker ticket is open while the linked PR is merged, the design doc is marked shipped while the deploy log shows no release. Silently picking one reading and presenting it as "the status" hides the disagreement and lets the bird-dogger believe in a status that half the evidence contradicts.

R006 names the *what*: when two or more sources report different statuses for the same item, the disagreement is visible alongside the status, not collapsed. R006 does not prescribe a resolution rule, a tie-breaker, or whether to compute a single "best guess" status — the requirement is honesty that sources disagree.

Conflict is shown via the basis (R001) and reduces confidence (R003). Override-vs-source surfacing — when an override (R004) is in effect and the inferred reading from sources disagrees with it — is not R006's scope; R004 owns it, surfacing the disagreement alongside the override rather than silently replacing or silently aligning.

## Edge Cases

- Single source: no conflict possible; no flag, no false-positive disagreement.
- Sources agree but each carries weak evidence: not a conflict; thinness is carried by R003 confidence, not by R006.
- Override (R004) disagrees with current source readings: out of R006's scope; R004 surfaces this disagreement and protects the override. R006 does not flag such items as multi-source conflicts.

## Examples

### PR merged, ticket still open

- Input: tracker ticket "open"; linked PR "merged" yesterday; no override.
- Expected Output: status flagged as conflicting at the list level; basis (R001) shows both readings; confidence (R003) reflects the disagreement; bird-dogger decides whether to override (R004) or investigate.
- Verification: the disagreement is visible without opening either source.

## Acceptance Criteria

- [ ] When sources disagree on an item's status, the disagreement is surfaced at the list level.
- [ ] The product does not silently resolve a multi-source conflict to a single value.
- [ ] Single-source items are not flagged as conflicting.

## Dependencies

- J001-O003-R001 — conflicts are surfaced through the basis trace.
- J001-O003-R003 — conflict lowers inferred confidence.

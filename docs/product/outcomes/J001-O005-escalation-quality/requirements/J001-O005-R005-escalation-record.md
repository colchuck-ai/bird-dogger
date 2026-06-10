# J001-O005-R005 - Escalation Record

The product must record each escalation attempt — target, channel,
time, and response — and surface that history at the item level.

## Detail

Without a persisted history, the bird-dogger re-escalates paths
that already failed or skips steps in the chain, because prior
attempts are not carried between checkpoints (RSK004).

R005 names the *what*: each escalation attempt is recorded with
target (who on R001's chain), channel (e.g., DM, email, meeting),
time, and response (received, none yet, declined, redirected).
That history is surfaced at the item level. R005 does not
prescribe the channel taxonomy, the response state vocabulary, or
the retention horizon — those are engineering-layer. Routine
touches the bird-dogger uses to anchor R003's response window
follow the same recording shape; the distinction between
"escalation attempt" and "touch" is a label on the record, not a
separate store.

R005 is the memory hub for O005. R003 reads the most recent touch
to compute its window; R004 reads prior attempts as a synthesis
input. R005 does not read R003 or R004 of the same cycle.

R005 is the O005 analogue of O004-R004 (triage state
carry-forward), but the shape differs: R005 records a list of
events (multiple attempts on the same item over time), not a
single current decision. The "what has changed since" anchoring
that O004-R004 uses is not R005's primary job — R005's primary
job is preserving the attempt log itself.

## Edge Cases

- Escalation attempt made but the channel did not deliver
  (bounced email, deleted message): recorded with the failure
  state explicit per
  [PRD001](../../../pdrs/PRD001-coverage-over-precision.md), not
  recorded as a successful attempt.
- Response received via a channel different from the attempt
  (escalated by DM, responded in a meeting): recorded as a
  response linked to the attempt; the channel mismatch is
  preserved, not normalized.
- Attempt made before R001's chain was registered (no recorded
  target): the attempt is still recorded; the target field
  surfaces as "unrecorded target" per PRD001 rather than dropping
  the attempt from history.

## Examples

### Re-escalation prevention

- Input: an item was escalated to the owner's manager two
  checkpoints ago via DM with no response; the bird-dogger is
  reviewing the item now.
- Expected Output: history view shows the prior attempt with
  target, channel, time, and "no response"; R004's synthesis
  reads that history and recommends a different target on the
  chain rather than the same one again.
- Verification: the bird-dogger does not re-send the same DM to
  the same target without seeing the prior failed attempt first.

## Acceptance Criteria

- [ ] Each escalation attempt is recorded with target, channel,
      time, and response state.
- [ ] The escalation history is visible at the item level across
      checkpoints, not only within the checkpoint where the
      attempt was made.
- [ ] Failed-delivery attempts are recorded with the failure
      state explicit rather than omitted or normalized.

## Dependencies

- (none intra-outcome — R005 is the memory hub; R003 and R004
  consume it, not the reverse)
- J001-O005-R001 — recorded targets are members of the chain
  R001 surfaces; R005 stores which target on the chain each
  attempt landed on.

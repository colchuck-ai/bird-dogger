# J001-O004-R002 - List-Level Urgency Signals

The product must show the facts that drive each item's urgency at
the list level, not only on the item's detail view.

## Detail

The bird-dogger triages from the list. Forcing a click into each
item to recognize what's urgent reintroduces the per-item
inspection cost the outcome exists to eliminate.

R002 names the *what*: urgency-driving signals visible alongside
the items being triaged. The set includes slip indicators
(O001), status basis and confidence (O003), per-item freshness,
due-date proximity, blocker age — the signals named in the
RSK002 description.

R002 does not prescribe density, layout, or which signal
dominates the row. It does commit to two shape rules from the
PDRs: per-item freshness follows the two-timestamp shape per
[PRD002](../../../pdrs/PRD002-per-item-freshness-presentation.md),
and missing or unassessable signals are surfaced explicitly per
[PRD001](../../../pdrs/PRD001-coverage-over-precision.md) rather
than blanked.

## Edge Cases

- A key urgency signal cannot be assessed for an item (source
  unreachable, never assessed, conflicting sources): surfaced
  explicitly per PRD001, not omitted from the row.
- Item carries many active signals: signals readable without
  burying the dominant one; R002 commits to "at the list level"
  not to a specific density.
- Source does not expose an underlying-data-change timestamp:
  the data-change time surfaces as "unknown" per PRD002/PRD001,
  not blank or set equal to the assessment time.

## Examples

### Triage list with mixed signal states

- Input: 8 items; 2 with full signal sets (slip + status + recent
  freshness), 3 with stale data-change timestamps, 2 with
  unreachable sources, 1 newly registered with no assessment
  yet.
- Expected Output: each row shows enough signals for the
  bird-dogger to recognize urgency; the 2 unreachable-source
  items show the unreachability inline rather than reading as
  "fine"; the newly-registered item shows "not yet assessed"
  rather than blank.
- Verification: the bird-dogger can pick the next intervention
  target from the list view without opening any item.

## Acceptance Criteria

- [ ] Each item shows enough signals at the list level to
      differentiate its urgency without opening the item view.
- [ ] Items with missing or unassessable signals are
      distinguishable from items with present-but-weak signals.
- [ ] Per-item freshness on the list view carries both
      timestamps per PRD002 — assessment time and underlying
      data-change time, distinct.

## Dependencies

- J001-O004-R001 — R001 defines what counts as urgency; R002
  surfaces those signals at the list level.
- J001-O001-R002 — per-item assessment freshness is one of the
  signals R002 surfaces (slip-side cousin under PRD002).
- J001-O003-R001 — status basis is one of the urgency signals;
  R002 surfaces the basis summary at the list level rather than
  forcing a drill-down.
- J001-O003-R002 — per-item status freshness is one of the
  signals R002 surfaces (status-side cousin under PRD002).

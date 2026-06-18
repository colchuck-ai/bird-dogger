# ADR008 - History Posture per Record Type

The CLI uses two removal postures depending on record type:
`override remove` deactivates the record and preserves its history per
[PRD004](../../product/pdrs/PRD004-override-surfacing.md); `touch remove`
and `note remove` hard-delete the record because corrective records
carry no equivalent history obligation.

## Context

The CLI manages two categories of non-declarative records beyond the
entity graph — records that capture intent or events at a point in time:

- **Overrides** (`bdog override add`) record a temporary policy
  adjustment against an item: why the bird-dogger's judgment differs from
  the automated assessment. They have a `--reason` and an `--expires`
  field. They affect how items are surfaced and triaged, and PRD004
  commits to surfacing their history (active and deactivated) so the
  bird-dogger can see how an item's override posture has evolved.

- **Touches** (`bdog touch add`) log a contact event — the fact that a
  specific contact was reached, when, and what was communicated. A touch
  is an append-only record of something that happened.

- **Notes** (`bdog note add`) produce a first-class C012 triage record
  with an auto-captured context snapshot. Notes are the bird-dogger's
  structured commentary on an item at a moment in time.

When a bird-dogger runs `remove` on any of these records, two behaviors
are possible:

1. **Deactivate, preserve history.** The record is marked inactive but
   stays in the store. `list --all` surfaces it. No data is destroyed.
2. **Hard delete.** The record is permanently dropped from the store.
   `list` will never surface it again.

The choice is not uniform across record types. Overrides require history
because PRD004 commits to it and because a removed override that left no
trace makes past surfacing decisions unauditable. Touches and notes,
by contrast, are recorded to correct mistakes — a duplicate touch or a
mistyped note should be cleanly removable, not merely hidden behind an
`--all` flag that still shows the error in perpetuity.

Without a recorded decision, an implementer might default to
uniform hard-delete (the simplest code path) and silently break PRD004
for overrides, or implement uniform deactivate and make it impossible to
truly correct a touch or note error.

## Options

- **A. Uniform hard-delete for all three record types.** Simplest
  implementation — one path through the `remove` handler regardless of
  type. Cost: PRD004 is violated for overrides; override history
  disappears on `remove`; a bird-dogger who deactivates an override
  and later wants to review why the assessment changed has no record.

- **B. Uniform deactivate (soft-delete) for all three record types.**
  Override history is preserved. Cost: touches and notes are never truly
  removed — a bird-dogger who fat-fingered a touch recipient or attached
  a note to the wrong item can only hide the record, not remove it; the
  data-correction use case is permanently closed.

- **C. Differentiated by history obligation: deactivate for overrides,
  hard-delete for touches and notes.** Override history is preserved
  per PRD004; touches and notes are cleanly correctable. Cost: two
  distinct `remove` paths depending on type; the distinction must be
  documented clearly so future record types make a conscious choice.

## Decision

Option C — differentiated by history obligation.

Overrides are policy-layer records whose history is a PRD commitment, not
an implementation convenience. Silent deletion would produce gaps in the
override audit trail and give the bird-dogger no way to understand why an
item's surfacing posture changed over time. The deactivate posture costs
nothing in the happy path — the record is invisible unless `--all` is
passed — and satisfies PRD004 without special-casing.

Touches and notes are corrective records. A touch logs "this contact was
reached on this date with this message"; a note records a triage
observation. Both exist to correct or extend the record at a specific
moment. The history obligation runs forward (new records accumulate), not
backward (old records should be preservable even if erroneous). Hard
deletion restores a clean slate when the record was created by mistake.
Forcing the bird-dogger to live with a deactivated-but-visible
bad-recipient touch or mistyped note frustrates the correction use case
without adding auditable value.

The two-path implementation cost is low: the only behavioral difference
is whether the store drops the row or sets `active = false`. The
distinction is documented in the CLI design doc and here, so future
record types have a clear decision framework: does PRD/product commit to
history for this type? If yes, deactivate. If no, hard-delete.

## Consequences

- **`override remove`** marks `active = false` on the override record.
  The record is not dropped. `bdog override list` returns only active
  overrides by default; `bdog override list --all` includes deactivated
  history. `override remove` is idempotent: removing an
  already-deactivated override succeeds as a no-op; history is preserved
  either way.

- **`touch remove`** permanently drops the touch record from the store.
  Not idempotent: removing a non-existent or already-removed touch errors
  with `not found`.

- **`note remove`** permanently drops the note record from the store.
  Not idempotent: same as touch.

- The removal posture for a record type is determined at design time, not
  at call time. There is no `--hard` or `--soft` flag on `remove`; the
  behavior is fixed per type.

- Adding a new non-declarative record type requires a posture decision
  before the `remove` handler is written. The default assumption should
  be hard-delete unless a product/PRD commitment to history exists for
  that type.

- See [cli.md](../design-docs/cli.md) "Removal semantics" for the prose
  summary of both postures and the idempotence table for the
  type-specific behavior.

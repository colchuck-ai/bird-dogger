# Override Store (C010)

One persistent override mechanism, parameterized by which produced judgment it covers — status, intervention recommendation, and escalation timing recommendation — with composite identity `(item, field)`, deactivate-on-remove history, and active overrides that survive re-synthesis.

## Data model

SQLite-backed State-band store. One table holds all override records; the `field` column selects which produced judgment the row corrects. Realizes [PRD004](../../product/pdrs/PRD004-override-surfacing.md).

### `overrides(item_id, field, value, reason, active, set_at, updated_at)`

| Column | Role |
|--------|------|
| **`item_id`** | References Item Store (C007) `items.id` (`bdogitem-<n>`). |
| **`field`** | Which produced judgment this override covers — `status`, `intervention`, or `escalation-timing`. |
| **`value`** | Closed-vocabulary override state for the field (see *Value vocabularies* below). |
| **`reason`** | Optional bird-dogger justification (`--reason` on write); free text. |
| **`active`** | `true` while the override is in force; `false` after `override remove` (deactivated, not dropped). |
| **`set_at`** | When the override was first established on this `(item_id, field)` pair. |
| **`updated_at`** | When `value`, `reason`, or `active` last changed. |

### Composite key

- **Primary key:** `(item_id, field)`.
- At most one row per item per judgment field. `status`, `intervention`, and `escalation-timing` are independent — an item may carry up to three active overrides simultaneously.
- Overrides have no standalone native id; identity is the positional pair `(<item>, <field>)` at the CLI surface ([C001](C001-cli.md#override)).

### Value vocabularies

| `field` | Produced by | Allowed `value` tokens |
|---------|-------------|------------------------|
| `status` | Assessment Engine (C009) | Source-type status vocabulary for source-derived items; freeform for manual items ([C001](C001-cli.md#override)). |
| `intervention` | Synthesis Engine (C011) | `intervene-now`, `not-now`, `cannot-recommend` |
| `escalation-timing` | Synthesis Engine (C011) | `escalate-now`, `not-yet`, `cannot-recommend` |

Recommendation-layer fields use closed enums so the override has a stable target and list rendering stays readable ([PRD004](../../product/pdrs/PRD004-override-surfacing.md), J001-O004-R003, J001-O005-R004).

### Invariants

- **Active override survives re-synthesis.** Assessment Engine (C009) and Synthesis Engine (C011) never write C010. Re-assessment and re-synthesis produce fresh readings; Override Store (C010) holds the bird-dogger's correction until cleared. Readers compose override + produced output at render time ([PRD004](../../product/pdrs/PRD004-override-surfacing.md)).
- **Deactivate on remove, preserve history.** `override remove` sets `active = false`; the row remains for `--all` listing per [ADR008](../adrs/ADR008-history-posture-per-record-type.md).
- **Item foreign key.** Every `item_id` must resolve to an existing Item Store (C007) row at write time.
- **No cascade on item scope loss.** When an item falls out of hunt scope, override rows remain attached to the item id ([ADR007](../adrs/ADR007-no-cascade-removal.md)).

## Interfaces

Override Store (C010) has no standalone CLI surface. [CLI (C001)](C001-cli.md#override) dispatches override verbs; downstream engines and Renderer (C015) read through internal contracts.

### Write path (via CLI → C010)

| CLI verb | Effect on C010 |
|----------|----------------|
| `bdog override add <item> <field> <value> [--reason <text>]` | Insert row with `active = true`; errors if `(item, field)` already exists. |
| `bdog override set <item> <field> <value> [--reason <text>]` | Update `value` and optional `reason` on existing row; reactivates if previously deactivated. |
| `bdog override remove <item> <field>` | Set `active = false`; row retained. Idempotent no-op if already inactive. |

### Read path

| Consumer | Reads |
|----------|-------|
| CLI (C001) | `info`, `list` with filters `--item`, `--field`, `--all` (includes deactivated history). |
| Assessment Engine (C009) | Active `status` overrides when composing assessment display inputs; does not overwrite C010. |
| Synthesis Engine (C011) | Active overrides for all three fields when ranking and recommending; produced output is separate from stored overrides. |
| Renderer (C015) | Active overrides for list and detail surfaces; disagreement marker when override value differs from current inferred reading ([PRD004](../../product/pdrs/PRD004-override-surfacing.md)). |
| Note Store (C012) | Active overrides at note write time for context snapshot capture (read-only at note `add`). |

## Behavior

### Establishing an override

On `override add`, CLI (C001) validates the item exists in Item Store (C007), validates `field` and `value` against the vocabulary for that field, and inserts a row with `active = true`. The bird-dogger's override becomes the displayed value for that judgment until cleared; Assessment Engine (C009) and Synthesis Engine (C011) continue to produce their own readings on each bugel pass without mutating C010.

### Updating an override

`override set` changes `value` and/or `reason` on an existing `(item, field)` row. If the row was previously deactivated, `set` reactivates it (`active = true`) with the new value — the bird-dogger is re-applying a correction without requiring a separate `add` after `remove`.

### Clearing an override

`override remove` deactivates the row per [ADR008](../adrs/ADR008-history-posture-per-record-type.md). The displayed value reverts to the current produced reading from Assessment Engine (C009) or Synthesis Engine (C011). Deactivated rows remain queryable via `override list --all` or item detail with `--show-history`.

| Re-invocation | Behavior |
|---------------|----------|
| `override remove` on active row | Sets `active = false`. |
| `override remove` on already-inactive row | Idempotent no-op; history preserved. |

### Surviving re-synthesis

Bugel re-runs assessment and synthesis on each checkpoint. Override Store (C010) is not consulted for writes during those passes — only for reads at composition time. When a new produced reading disagrees with an active override:

1. The override value remains the displayed value.
2. The fresh inferred reading is surfaced alongside (Renderer (C015) parenthetical + `*` marker per [PRD004](../../product/pdrs/PRD004-override-surfacing.md)).
3. The bird-dogger adopts the new reading (typically by updating or clearing the override via CLI) or dismisses it by leaving the override in place.

No automatic adoption or expiry on re-synthesis.

### Listing and filtering

- **Default `override list`:** active rows only (`active = true`).
- **`--all`:** active and deactivated rows — full history for an audit of override posture over time ([ADR008](../adrs/ADR008-history-posture-per-record-type.md), [PRD004](../../product/pdrs/PRD004-override-surfacing.md)).
- **`--item` / `--field`:** narrow to one item or one judgment field.

## Edge cases

- **Re-assessment disagrees with active status override:** Override stands; inferred status surfaces alongside; override is not silently replaced (J001-O003-R004).
- **Re-synthesis disagrees with intervention or escalation-timing override:** Same PRD004 shape as status — override displayed, new recommendation in parentheses when different.
- **Override cleared:** Display reverts to current produced reading; deactivated row remains in history (`--all`, `--show-history`).
- **Manual item with status override:** Override may be the only status the item carries; same store path as source-backed items (J001-O003-R004).
- **`override add` on existing `(item, field)`:** Errors on duplicate identity; use `set` to change value or reactivate after `remove`.
- **Three independent fields:** Status, intervention, and escalation-timing overrides compose — clearing one field does not affect the others.
- **Item falls out of scope:** Override rows remain on the item; readable when the item id is known ([ADR007](../adrs/ADR007-no-cascade-removal.md)).
- **Produced state is `cannot-recommend`:** Override can still target the field; PRD001 uncertainty and PRD004 authority compose without either silently overwriting the other.

## Relationships

- **CLI (C001)**: sole writer — dispatches `override add`, `set`, `remove`, `info`, `list`.
- **Item Store (C007)**: `item_id` foreign key; every override belongs to one item.
- **Assessment Engine (C009)**: produces status readings; reads active status overrides at composition time; never writes C010.
- **Synthesis Engine (C011)**: produces intervention and escalation-timing recommendations; reads active overrides at synthesis/render composition; never writes C010.
- **Renderer (C015)**: reads overrides for bugel table, item detail, and override CRUD surfaces; applies PRD004 disagreement rendering.
- **Note Store (C012)**: reads active overrides when capturing note context snapshots — sibling State-band store; notes are open-vocabulary commentary, not judgment corrections.
- **Touch Log (C013)**: sibling State-band store; contact events, not overrides. Removal posture differs (hard-delete vs deactivate) per [ADR008](../adrs/ADR008-history-posture-per-record-type.md).

## Success criteria

- Doc describes one parameterized mechanism for status, intervention, and escalation-timing — not three separate stores.
- Composite key `(item_id, field)` documented; no standalone override id.
- Active override survives re-synthesis documented; engines read but do not write C010.
- `override remove` deactivates (`active = false`) and preserves history; `override list --all` includes deactivated rows per [ADR008](../adrs/ADR008-history-posture-per-record-type.md).
- PRD004 disagreement surfacing traceable: override as displayed value, inferred reading alongside when different.
- Closed vocabulary per field documented; links to J001-O003-R004, J001-O004-R003, J001-O005-R004.

## Notes

### Design decisions

**One store, three fields ([engineering README](../README.md), [PRD004](../../product/pdrs/PRD004-override-surfacing.md)).** Status, intervention recommendation, and escalation timing recommendation share one table and one CLI verb family. The `field` column parameterizes which produced judgment is overridden. Separate stores would duplicate deactivate/history semantics and break the PRD004 commitment to one override mechanism.

**Composite key, not surrogate id ([C001](C001-cli.md#data-model)).** The bird-dogger addresses overrides as `(item, field)` — matching how they think about corrections ("override status on this item"). History for a field is the lifecycle of that one row (active ↔ inactive ↔ reactivated via `set`).

**Deactivate vs hard-delete ([ADR008](../adrs/ADR008-history-posture-per-record-type.md)).** Overrides are policy-layer records with a PRD history commitment. Touches and notes hard-delete on remove because they correct recording mistakes; override history explains how an item's surfacing posture evolved.

**Engines produce; C010 persists authority.** Assessment and synthesis outputs are ephemeral per checkpoint until rendered. The override is durable bird-dogger state that outlasts any single pass — the core of Option A in PRD004.

**Distinct from Note Store (C012).** Overrides use closed vocabulary tied to produced judgments. Notes use open text and capture a context snapshot at write time. Both are per-item State-band history but serve different jobs (judgment correction vs triage commentary).

## Open decisions

- **Adopt/dismiss affordance.** PRD004 names adopt-or-dismiss when disagreement is surfaced; v1 expresses dismissal by leaving the override in place and adoption by `override set` or `override remove`. A dedicated adopt verb (copy inferred reading into override or clear override in one step) remains CLI design territory under [C001](C001-cli.md).
- **`--expires` field.** [ADR008](../adrs/ADR008-history-posture-per-record-type.md) prose mentions an optional expiry on overrides; [C001](C001-cli.md#override) does not expose it. Whether expiry is in scope for v1 or deferred is unset.
- **Status vocabulary validation.** Source-derived items may use source-type status tokens; the exact validation rule (strict enum vs adapter-reported vocabulary) is open at the Assessment Engine / Source Adapter boundary.
- **Recommendation override basis trace.** Status cleared overrides are recoverable in basis history (J001-O003-R004); equivalent trace shape for intervention and escalation-timing cleared overrides is referenced by PRD004 but not yet specified in a component-level ADR.

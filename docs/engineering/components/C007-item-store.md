# Item Store (C007)

Schema authority for the State band — owns per-item facts, materialized scope edges, assessment readings, and the `active_set(H)` query every downstream component uses to enumerate hunt membership.

## Data model

SQLite-backed. Item Store (C007) is the identity hub for the State band: every other State-band store keys into items by `bdogitem-<n>` per [ADR006](../adrs/ADR006-native-id-scheme.md). Declarative config (sources, selectors, hunts) lives in Config Store (C002); Item Store (C007) holds produced state only.

### SQLite table inventory

| Table | Owner | Write site | Purpose |
|-------|-------|------------|---------|
| `items` | Item Store (C007) | CLI (C001), Refresh Engine (C008) | Core per-item row: identity, origin, title, trajectory, owner/chain refs, underlying-data timestamp |
| `item_selectors` | Item Store (C007) | Refresh Engine (C008) only | Source-backed scope junction `(item_id, selector_id)` |
| `item_hunts` | Item Store (C007) | CLI (C001) only | Manual multi-hunt scope junction `(item_id, hunt_id)` |
| `item_depends_on` | Item Store (C007) | CLI (C001) only | Directed dependency edges between items |
| `item_assessment_readings` | Item Store (C007) | Assessment Engine (C009) only | Latest slip and status readings per item (includes last-assessed timestamp per [PRD002](../../product/pdrs/PRD002-per-item-freshness-presentation.md)) |
| `item_basis_traces` | Item Store (C007) | Assessment Engine (C009) only | Evidence and conflict detail backing each assessment reading |

Hunt Refresh State (C016), Override Store (C010), Note Store (C012), Touch Log (C013), and Contact Registry (C014) hold their own tables and reference `items.id` — they do not duplicate item facts. See each component doc for its table list; **item id keys always mean `bdogitem-<n>` from this store.**

### `items`

| Field | Required | Notes |
|-------|----------|-------|
| `id` | yes | Native id `bdogitem-<n>`; assigned on first capture (manual) or first refresh (source-derived) |
| `origin` | yes | `manual` or `source-derived` — governs which scope junctions apply |
| `title` | yes | Display title; manual from CLI; source-derived from adapter pull |
| `source_id` | source-derived only | Source **id** (e.g. `bdogsource-1`) for source-derived items; absent on manual items |
| `source_item_id` | source-derived only | Source-side alias (e.g. `SUPPLY-1234`); CLI convenience only — native id is durable key |
| `trajectory_kind` | yes | `due-date`, `pace`, or `none` |
| `due_date` | when `due-date` | Expected completion date |
| `pace` | when `pace` | Expected pace descriptor |
| `owner_contact_id` | optional | Contact **id** from Config Store (C002) |
| `chain_id` | optional | Chain **id** from Config Store (C002) |
| `underlying_data_changed_at` | optional | Per-item "last underlying data change" timestamp ([PRD002](../../product/pdrs/PRD002-per-item-freshness-presentation.md)); written by Refresh Engine (C008) on refresh; literal unknown surfaced when source provides no signal |
| `created_at` | yes | First capture time |

**Id assignment.** Config Store (C002) owns the monotonic `bdogitem-<n>` counter. CLI (C001) assigns on manual `item add`; Refresh Engine (C008) assigns on first upsert of a source-derived item. Sequence gaps from deletion are tolerated; ids are never reused ([ADR006](../adrs/ADR006-native-id-scheme.md)).

**Origin tag invariants.**

- `manual`: no `source_id`, no `source_item_id`; may have zero or more `item_hunts` rows; never receives `item_selectors` rows.
- `source-derived`: has `source_id` and usually `source_item_id`; scope enters only via `item_selectors` reconciliation; never receives `item_hunts` rows.

### `item_selectors(item_id, selector_id)`

- **Primary key or unique constraint:** `(item_id, selector_id)`.
- `item_id` → `items.id` (`bdogitem-<n>`).
- `selector_id` → Selector Registry (C017) selector **id** (`bdogselector-<n>`).
- Optional `matched_at` — when the link was last confirmed by a pull (debugging).
- **Write site:** Refresh Engine (C008) exclusively, during per-selector reconciliation.
- **Origin constraint:** source-derived items only.

### `item_hunts(item_id, hunt_id)`

- **Primary key or unique constraint:** `(item_id, hunt_id)`.
- `item_id` → `items.id` (`bdogitem-<n>`).
- `hunt_id` → Hunt Registry (C006) hunt **name** (CLI positional key).
- **Write site:** CLI (C001) via `item hunt add` / `item hunt remove` and optional repeatable `--hunt` on `item add`.
- **Origin constraint:** manual items only.

### `item_depends_on(item_id, depends_on_item_id)`

- Directed edge: `item_id` depends on `depends_on_item_id`.
- Both columns reference `items.id`.
- **Write site:** CLI (C001) via `item depends-on` verbs.
- No cascade delete: removing an item requires clearing dependents first ([ADR007](../adrs/ADR007-no-cascade-removal.md)).

### `item_assessment_readings`

Latest produced readings Assessment Engine (C009) writes back after each assessment pass. At minimum:

| Reading | Fields (conceptual) | Notes |
|---------|-------------------|-------|
| Slip reading | slip status, trajectory comparison, silence signal, `assessed_at` | O001 slip detection |
| Status reading | status value, confidence, conflict flag, `assessed_at` | O003 status fidelity |

**Freshness split ([PRD002](../../product/pdrs/PRD002-per-item-freshness-presentation.md)).** `assessed_at` on the assessment reading is the per-item "last assessed" timestamp. `items.underlying_data_changed_at` is the distinct "last underlying data change" timestamp Refresh Engine (C008) maintains. These two values are never collapsed into one field.

### `item_basis_traces`

Structured evidence backing each assessment reading — source contributions, disagreement detail, override interaction, and "cannot assess" / "sources disagree" room per [PRD001](../../product/pdrs/PRD001-coverage-over-precision.md). Written by Assessment Engine (C009); read by Renderer (C015) on `--show-basis` and detail surfaces.

### Canonical `active_set(H)` query

Verbatim from [ADR012](../adrs/ADR012-scope-via-selectors.md):

```
active_set(H) =
  { item | ∃ selector S : (item, S) ∈ item_selectors ∧ S ∈ hunt(H).selectors }
  ∪
  { item | (item, H) ∈ item_hunts }
```

Where `hunt(H).selectors` is the set of selector **ids** referenced by hunt H, resolved to names via Selector Registry (C017) and Hunt Registry (C006) for display only — the junction stores selector **ids**.

Implementation dedupes by `items.id` when an item matches multiple selectors in the same hunt or appears in both branches of the union.

### Invariants

- At most one `items` row per native id; at most one row per `(source_id, source_item_id)` pair for source-derived items.
- Manual items never have `item_selectors` rows; source-derived items never have `item_hunts` rows.
- Items may exist with **zero** scope links — no `item_selectors` and no `item_hunts` rows — and remain in Item Store (C007) with full history ([ADR007](../adrs/ADR007-no-cascade-removal.md)).
- Item enumeration for hunt H always uses `active_set(H)` here — never Hunt Refresh State (C016) membership reads.
- Cross-component item references use `bdogitem-<n>` only on disk; CLI accepts source aliases when unambiguous ([ADR006](../adrs/ADR006-native-id-scheme.md)).

## Interfaces

Item Store (C007) has no CLI surface of its own. CLI (C001) dispatches `item` verbs; Item Store (C007) validates, persists, and resolves identity while other engines read and write through internal contracts.

### CLI dispatch (reference only)

Full verb shapes live in [CLI (C001)](C001-cli.md#item). Summary of what crosses into Item Store (C007):

| CLI verb | Item Store (C007) role |
|----------|------------------------|
| `item add` | Assign `bdogitem-<n>`; insert `items` row with `origin = manual`; optional repeatable `--hunt` seeds `item_hunts` |
| `item remove` | Reject if dependents exist; delete manual item row and its junction/reading rows |
| `item info` / `list` | Read item facts, scope links, readings; `list --hunt` uses `active_set(H)` |
| `item set` | Update title, trajectory, owner, chain on `items` |
| `item hunt add/remove/list` | Mutate `item_hunts` for manual items only |
| `item depends-on add/remove/list` | Mutate `item_depends_on` |

### Read

| Caller | Trigger | Item Store (C007) action |
|--------|---------|--------------------------|
| CLI (C001) | `item info`, `list`, chase verbs | Load items; resolve `<item>` native or alias |
| Refresh Engine (C008) | Refresh upsert | Lookup by `(source_id, source_item_id)`; enumerate links for reconciliation |
| Assessment Engine (C009) | Assessment pass | Read per-item facts, `active_set(H)` per hunt, underlying-data timestamps |
| Synthesis Engine (C011) | Synthesis pass | Read facts and `active_set(H)` |
| Renderer (C015) | Bugel, `item list`, detail | Read facts, readings, `active_set(H)` for row sets |
| Hunt Registry (C006) | `hunt remove` pre-check | List manual items with `item_hunts` rows for hunt |
| Override Store (C010), Note Store (C012), Touch Log (C013) | Per-item CRUD | Verify `items.id` exists |

### Write

| Caller | Trigger | Item Store (C007) action |
|--------|---------|--------------------------|
| CLI (C001) | `item add`, `set`, `remove`, `item hunt`, `item depends-on` | Mutate `items`, `item_hunts`, `item_depends_on` per origin rules |
| Refresh Engine (C008) | Selector pull + reconciliation | Upsert source-derived `items`; update `underlying_data_changed_at`; reconcile `item_selectors` per selector |
| Assessment Engine (C009) | Post-assessment | Upsert `item_assessment_readings` and `item_basis_traces` |

### Identity resolution contract

```
resolve_item(ref) → items.id | error
```

- **Native id:** direct lookup on `bdogitem-<n>`.
- **Source alias:** match `source_item_id` when exactly one item maps; ambiguous match lists all `bdogitem-<n>` candidates and fails fast ([ADR006](../adrs/ADR006-native-id-scheme.md)).
- **Not found:** absent from `items`.

### Active-set query contract

```
active_set(hunt_name) → [items.id, ...]   # deduped
active_set_count(hunt_name) → integer
```

- Resolves hunt via Hunt Registry (C006); expands selector ids; unions junction branches per ADR012 definition above.
- Used by Assessment Engine (C009), Synthesis Engine (C011), Renderer (C015), and CLI (C001) `item list --hunt`.

## Behavior

### Manual item lifecycle

1. CLI (C001) receives `item add` with required `--title` and optional trajectory, owner, chain, repeatable `--hunt`.
2. Item Store (C007) assigns `bdogitem-<n>`, inserts `items` with `origin = manual`.
3. Each supplied hunt seeds an `item_hunts` row; omitting `--hunt` creates an item with no hunt membership until `item hunt add`.
4. Trajectory cross-validation (`due-date` requires `--due`, etc.) runs before persist ([CLI (C001)](C001-cli.md#item)).

Manual items carry no source reference. Hunt membership is adjusted only via `item hunt` verbs — not via selector edits.

### Source-derived item lifecycle

1. Refresh Engine (C008) pulls items for a resolved selector triple via Source Adapter (C005).
2. For each returned source item, Item Store (C007) upserts `items` with `origin = source-derived`, assigns native id on first sight, sets title and source alias fields, adopts starting expected trajectory from source-side due date when present, and updates `underlying_data_changed_at` when the source signals change.
3. Reconciliation for selector S: **add** `(item_id, S)` for every returned item not already linked; **remove** `(item_id, S)` for existing links where the item was not in this pull's result set.
4. Reconciliation is per-selector, not per-hunt — one pull updates scope for all hunts referencing S ([ADR012](../adrs/ADR012-scope-via-selectors.md)).

Source-derived items never receive `item_hunts` rows. Scope adjustment is by editing selectors or hunt selector membership, not `item hunt` verbs.

### Assessment write-back

Assessment Engine (C009) reads `active_set(H)` and per-item facts, computes slip and status readings, then writes:

- `item_assessment_readings` — latest readings including `assessed_at`.
- `item_basis_traces` — evidence detail for `--show-basis` and conflict surfacing.

Item Store (C007) does not interpret readings; it persists what Assessment Engine (C009) produces. Re-assessment on underlying-data change ([J001-O003-R005](../../product/outcomes/J001-O003-status-fidelity/requirements/J001-O003-R005-re-assessment-on-change.md)) is triggered by Refresh Engine (C008) + Assessment Engine (C009) — Item Store (C007) supplies the timestamp comparison inputs only.

### Retention when scope drops

When reconciliation removes the last `item_selectors` link for an item, or the bird-dogger removes all `item_hunts` links, the `items` row and all dependent State-band history (notes, overrides, touches, readings) remain. The item simply falls out of every `active_set(H)` until re-linked or manually re-assigned ([ADR007](../adrs/ADR007-no-cascade-removal.md)).

### `--no-refresh` interaction

A bugel with `--no-refresh` skips Refresh Engine (C008) writes. Assessment and synthesis run against the last materialized `item_selectors` links and existing `items.underlying_data_changed_at` values. Selector declaration edits in Config Store (C002) are not reflected in scope until the next full refresh ([ADR012](../adrs/ADR012-scope-via-selectors.md) D5).

## Edge cases

**Item matches two selectors in one hunt.** One `items` row, two `item_selectors` rows. `active_set(H)` dedupes by item id — the item appears once.

**Shared selector across hunts.** One selector pull reconciles `item_selectors` once; every referencing hunt's `active_set` picks up the change via shared links without duplicate membership storage.

**Manual item in zero hunts.** Valid after `item hunt remove` clears all links or when `item add` omits `--hunt`. Item retained in Item Store (C007); appears in no hunt's active set.

**Manual item in multiple hunts.** `item hunt add` inserts `(item_id, hunt_id)` rows idempotently. Removing from one hunt does not affect others.

**Source id alias collision.** Two items share the same source-side key across sources or within one source — CLI commands accepting `<item>` fail fast with both native ids listed ([ADR006](../adrs/ADR006-native-id-scheme.md)).

**Source-derived item rejected for `item hunt`.** CLI (C001) rejects `item hunt` verbs on source-derived items; scope is selector-driven only.

**Hunt remove blocked by manual links.** Hunt Registry (C006) queries Item Store (C007) for `item_hunts` rows before delete; bird-dogger clears via `item hunt remove` or `item remove` ([ADR007](../adrs/ADR007-no-cascade-removal.md)).

**Item remove blocked by dependents.** `item depends-on` reverse lookup must be empty before `item remove`.

**Underlying-data timestamp unknown.** When Source Adapter (C005) provides no change signal, `underlying_data_changed_at` is absent or marked unknown; Renderer (C015) surfaces literal `unknown` per PRD002 — never copied from `assessed_at`.

**Selector edit before refresh.** `item_selectors` reflects last pull until reconciliation; `active_set(H)` may be stale relative to live selector declarations until next full bugel.

## Relationships

- **CLI (C001)**: dispatches manual item capture, trajectory/owner/chain updates, `item_hunts`, and `item_depends_on`; orchestrates user-visible identity and scope errors.
- **Config Store (C002)**: assigns `bdogitem-<n>` counters; owner/chain fields on `items` reference contact/chain ids from config.
- **Refresh Engine (C008)**: upserts source-derived items, writes `underlying_data_changed_at`, reconciles `item_selectors`; does not write `item_hunts` or assessment readings.
- **Assessment Engine (C009)**: reads facts and `active_set(H)`; writes assessment readings and basis traces.
- **Override Store (C010)**: references `items.id`; overrides sit alongside readings at render time — Item Store (C007) does not store overrides.
- **Synthesis Engine (C011)**: reads facts and `active_set(H)` at synthesis time.
- **Note Store (C012)**, **Touch Log (C013)**: reference `items.id` for per-item history.
- **Contact Registry (C014)**: owner/chain ids on `items` point at declared contacts/chains.
- **Hunt Registry (C006)**: supplies hunt names and selector id sets for `active_set(H)`; queries Item Store (C007) for hunt-remove pre-checks.
- **Selector Registry (C017)**: selector ids on `item_selectors` reference declared selectors.
- **Hunt Refresh State (C016)**: per-hunt refresh metadata only — never substitutes for `active_set(H)` enumeration from Item Store (C007).
- **Renderer (C015)**: reads item facts, readings, and active-set membership for list and detail surfaces.

## Success criteria

- Every SQLite table in the State-band item schema is listed above with a single owner and explicit write site per table.
- `active_set(H)` matches [ADR012](../adrs/ADR012-scope-via-selectors.md) verbatim — selector overlap ∪ manual `item_hunts`, deduped by item id.
- Manual items (`origin = manual`) never receive `item_selectors` rows; source-derived items never receive `item_hunts` rows.
- Items with zero scope links remain in Item Store (C007) with history intact when all junction rows are removed ([ADR007](../adrs/ADR007-no-cascade-removal.md)).
- Per-item freshness carries two distinct timestamps per [PRD002](../../product/pdrs/PRD002-per-item-freshness-presentation.md): `assessed_at` on assessment readings, `underlying_data_changed_at` on `items`.
- Override Store (C010), Note Store (C012), Touch Log (C013), and Contact Registry (C014) document item foreign keys as `bdogitem-<n>` referencing this store.
- Assessment Engine (C009) and Synthesis Engine (C011) enumerate hunt membership only through `active_set(H)` here — not through Hunt Refresh State (C016).
- Realizes **J001-O002-R001** (coverage set visibility via inspectable `active_set(H)`) and supports **J001-O003-R002**, **J001-O003-R005**, and **J001-O001-R002** freshness and re-assessment requirements through the two-timestamp shape.

## Notes

### Design decisions

**Schema authority.** Item Store (C007) is the single source of truth for item identity, per-item facts, materialized scope edges, and assessment artifacts. Other State-band components hold their own tables but never duplicate item rows or enumerate hunt membership independently.

**Dual junctions over unified scope table.** `item_selectors` and `item_hunts` separate refresh-materialized scope from CLI-materialized scope so write-site rules and origin constraints are table boundaries, not conventions ([ADR012](../adrs/ADR012-scope-via-selectors.md) option A).

**Native id as durable key.** Source aliases are CLI convenience; SQLite foreign keys and TOML cross-references use `bdogitem-<n>` only. Ambiguous aliases fail fast rather than silently picking one ([ADR006](../adrs/ADR006-native-id-scheme.md), [PRD001](../../product/pdrs/PRD001-coverage-over-precision.md)).

**No membership in Hunt Refresh State (C016).** Moving scope edges here eliminated duplicate `(hunt, item)` storage and enabled selector-deduped refresh. Set-level last-refresh remains in C016; item counts are `|active_set(H)|` queries against this store.

**Assessment artifacts co-located with facts.** Readings and basis traces live in Item Store (C007) because Assessment Engine (C009) is the sole writer and Renderer (C015) reads them alongside item facts for bugel rows. Override Store (C010) stays separate — overrides are bird-dogger judgments, not engine-produced readings.

### Worked example (active set union)

Hunt `supply-q3` references selectors `supply-p0` (`bdogselector-3`) and `supply-blockers` (`bdogselector-7`).

After refresh:

- `bdogitem-12` linked via `item_selectors` to `bdogselector-3` (source-derived).
- `bdogitem-42` linked via `item_selectors` to both selectors (two junction rows, one item row).
- `bdogitem-99` manual item with `item_hunts` row for `supply-q3`.

```
active_set(supply-q3) = { bdogitem-12, bdogitem-42, bdogitem-99 }
```

Removing `bdogitem-99` from the hunt via `item hunt remove` drops it from the active set but retains the item row for notes and history.

### Worked example (scope drop, item retained)

`bdogitem-12` stops matching selector `supply-p0` on the next refresh. Reconciliation removes `(bdogitem-12, bdogselector-3)`. The `items` row, notes, overrides, and assessment history remain. `bdogitem-12` is absent from `active_set(supply-q3)` until it matches again.

## Open decisions

- **Multi-source underlying-data picking rule.** [PRD002](../../product/pdrs/PRD002-per-item-freshness-presentation.md) commits to one per-item `underlying_data_changed_at` value when an item's basis spans multiple sources; the max-of vs per-source composite rule is ADR territory — not decided in this component doc.
- **Assessment reading schema detail.** Slip vs status column shapes, versioning of readings on re-assessment, and basis-trace serialization format will be specified when Assessment Engine (C009) component doc lands; Item Store (C007) owns the tables but defers field-level detail to that doc.
- **Trajectory pace semantics.** `pace` trajectory storage and comparison rules for slip detection depend on Assessment Engine (C009) and Source Adapter (C005) contracts — v1 docs record the field; comparison algorithm is not finalized here.

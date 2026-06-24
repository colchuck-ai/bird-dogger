# Hunt Refresh State (C016)

Owns per-hunt refresh metadata only — no `(hunt, item)` membership rows. Holds the set-level last-refresh marker and the per-hunt source-availability snapshot produced on each refresh. Prose uses **Hunt Refresh State (C016)**; the component id **C016** is unchanged from the pre-[ADR012](../adrs/ADR012-scope-via-selectors.md) "Coverage Memory" label. SQLite-backed; metadata-only reads for downstream Reasoning and Renderer components.

What a hunt currently includes is **never** stored or enumerated here. Scope membership is computed from Item Store (C007) junctions via `active_set(H)` per [ADR012](../adrs/ADR012-scope-via-selectors.md).

## Data model

Two tables. No `(hunt, item)` membership table exists or is planned.

### `hunt_refresh(hunt_id, refreshed_at)`

- **Primary key:** `hunt_id` (references Hunt Registry (C006) hunt name).
- **`refreshed_at`:** timestamp of the most recent full refresh that updated this hunt's metadata (set-level last-refresh marker).
- One row per hunt that has been refreshed at least once. Absent until the first refresh for that hunt.

### `hunt_source_availability(hunt_id, source_id, status, reason, refreshed_at)`

- **Primary key or unique constraint:** `(hunt_id, source_id)`.
- **`hunt_id`:** references Hunt Registry (C006) hunt name.
- **`source_id`:** references Source Registry (C004) source name.
- **`status`:** availability outcome for this source as seen from this hunt's refresh context (e.g. available vs unavailable per J001-O001-R006).
- **`reason`:** when unavailable, a machine-readable cause — e.g. `network-error`, `file-missing`, `unparseable`, `auth-failed` (see [Renderer (C015)](C015-renderer.md) bugel source-availability detail).
- **`refreshed_at`:** timestamp of the refresh pass that wrote this snapshot row (may match the hunt's `hunt_refresh.refreshed_at` for that pass).

Rows are keyed per `(hunt, source)` so a hunt that references multiple selectors on the same source carries one availability snapshot for that source on that hunt. Rows cover sources touched by the hunt's selectors during the refresh pass; sources not referenced by the hunt are absent.

### Invariants

- Hunt Refresh State (C016) does **not** store item ids, item counts, or membership edges.
- Item enumeration for hunt H is always `active_set(H)` in Item Store (C007) — selector overlap ∪ manual `item_hunts` — never a read from C016.
- Writes originate exclusively from Refresh Engine (C008) during refresh; CLI (C001) does not write C016 directly.

## Interfaces

Hunt Refresh State (C016) is a State-band store with no CLI surface of its own. Other components query it through internal read contracts.

### Read patterns (metadata only)

| Consumer | Reads | Does not read |
|----------|-------|----------------|
| Assessment Engine (C009) | Optional refresh metadata for freshness context | Item membership |
| Synthesis Engine (C011) | Optional hunt refresh metadata when freshness display needs it | Item enumeration |
| Renderer (C015) | Set-level `refreshed_at` for bugel header and `hunt info`; per-source availability for header summary and source-availability detail | Rows for the active-items table |

### Write patterns

| Writer | Writes |
|--------|--------|
| Refresh Engine (C008) | `hunt_refresh` and `hunt_source_availability` per affected hunt on each non-skipped refresh |

No other component writes Hunt Refresh State (C016).

## Behavior

### On refresh (`bdog hunt bugel` without `--no-refresh`)

After Refresh Engine (C008) completes selector-deduped pulls and `item_selectors` reconciliation in Item Store (C007), it updates Hunt Refresh State (C016) **per affected hunt** in the target set:

1. Upsert `hunt_refresh(hunt_id, refreshed_at)` with the refresh pass timestamp.
2. Upsert `hunt_source_availability` for each source touched by that hunt's selectors, carrying `status`, `reason` (when unavailable), and `refreshed_at`.

Refresh Engine (C008) does **not** write membership anywhere — including C016.

### On `--no-refresh`

Refresh Engine (C008) is not invoked. Hunt Refresh State (C016) is unchanged. Assessment, synthesis, and rendering use the last materialized refresh metadata and the last materialized `item_selectors` links in Item Store (C007).

### On `--dry-run`

Refresh Engine (C008) computes the delta to refresh metadata (and to `item_selectors`) without writing. Hunt Refresh State (C016) remains unchanged until a non-dry-run refresh commits.

### Relationship to `active_set(H)`

```
active_set(H) =
  { item | ∃ selector S : (item, S) ∈ item_selectors ∧ S ∈ hunt(H).selectors }
  ∪
  { item | (item, H) ∈ item_hunts }
```

Defined in [ADR012](../adrs/ADR012-scope-via-selectors.md) and owned by Item Store (C007). Hunt Refresh State (C016) answers "when did hunt H last refresh and which sources were available?" — not "which items does hunt H include?"

## Edge cases

**Shared selector across hunts.** One selector pull reconciles `item_selectors` once; each referencing hunt receives its own `hunt_refresh` and `hunt_source_availability` updates. Active-set changes are visible in every affected hunt via Item Store (C007) links, without duplicate membership storage in C016.

**Hunt never refreshed.** No `hunt_refresh` row; no availability rows. `hunt info` and bugel header show absent or unknown last-refresh semantics per Renderer (C015); active-item count still comes from `|active_set(H)|` in Item Store (C007).

**Selector edit before refresh.** Selector declaration changes are live in Config Store (C002) immediately; `item_selectors` and C016 metadata reflect the last pull until the next full refresh. A `--no-refresh` bugel intentionally uses stale scope edges and stale refresh metadata together.

**Source unavailable for one hunt's selectors.** `hunt_source_availability` records the failure for `(hunt, source)`. Items already linked via `item_selectors` may remain in `active_set(H)` but render as `source-unavailable` per J001-O001-R006 — membership enumeration still flows through Item Store (C007), not C016.

**Manual-only hunts.** Hunts whose selectors resolve only to manual items (no source pulls) still receive `hunt_refresh` updates when refresh runs; availability rows cover sources referenced by the hunt's selector set, not manual items.

**Multi-hunt bugel.** Each hunt in the target set gets independent C016 rows for the same refresh pass timestamp where applicable.

## Relationships

- **Refresh Engine (C008)**: sole writer — `hunt_refresh` and `hunt_source_availability` per affected hunt.
- **Item Store (C007)**: owns `item_selectors`, `item_hunts`, and the `active_set(H)` query; C016 never substitutes for C007 enumeration.
- **Hunt Registry (C006)**: `hunt_id` keys reference declared hunt names.
- **Source Registry (C004)**: `source_id` keys reference declared source names.
- **Assessment Engine (C009), Synthesis Engine (C011), Renderer (C015)**: may read refresh metadata; must enumerate items via Item Store (C007) `active_set(H)`, not C016.

## Success criteria

- Schema documentation covers `hunt_refresh(hunt_id, refreshed_at)` and `hunt_source_availability(hunt_id, source_id, status, reason, refreshed_at)` with no `(hunt, item)` table.
- Doc states explicitly that item enumeration is always via Item Store (C007) `active_set(H)`, never Hunt Refresh State (C016).
- Prose name "Hunt Refresh State" is used alongside component id C016.
- Write path is Refresh Engine (C008) only; read path is metadata-only for Assessment Engine (C009), Synthesis Engine (C011), and Renderer (C015).
- `hunt info` / bugel active-item counts are documented as `|active_set(H)|` from Item Store (C007), not C016 row counts.

## Notes

### Design decisions

**Slimmed role post-ADR012.** [ADR012](../adrs/ADR012-scope-via-selectors.md) moved materialized scope edges to Item Store (C007) (`item_selectors`, `item_hunts`). Hunt Refresh State (C016) retains only hunt-level refresh aggregates. The old "Coverage Memory" name reflected membership storage; the new prose name reflects the remaining responsibility.

**Per-hunt availability snapshot.** Source availability is stored per `(hunt, source)` so Renderer (C015) can show hunt-scoped availability in bugel header and `hunt info` without inferring hunt context from a global source-health table alone.

**Metadata-only downstream reads.** Assessment Engine (C009) and Synthesis Engine (C011) enumerate items through Item Store (C007). C016 contributes freshness and availability context only — consistent with [Renderer (C015)](C015-renderer.md) metadata-only reads for headers and counts.

## Open decisions

None — scope split, table shapes, write site, and enumeration rule are recorded in [ADR012](../adrs/ADR012-scope-via-selectors.md). Exact `status` enum values and Source Adapter (C005) → Refresh Engine (C008) handoff for availability remain product/engineering detail under J001-O001-R006; C015 documents renderer-facing reason strings.

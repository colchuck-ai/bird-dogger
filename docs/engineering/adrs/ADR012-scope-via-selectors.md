# ADR012 - Scope via Selectors

Coverage scope is materialized in Item Store (C007) as two junction tables — `item_selectors` for source-backed scope and `item_hunts` for manual multi-hunt assignment — instead of `(hunt, item)` membership rows in Coverage Memory (C016). Refresh Engine (C008) reconciles selector links selector-deduped across hunts; Coverage Memory (C016) holds hunt refresh metadata only.

## Context

[ADR001](ADR001-source-hunt-decoupling.md) established that one item surfaced in two hunts is one row in Item Store (C007) and two membership facts in Coverage Memory (C016). [ADR004](ADR004-selectors-as-first-class-declarations.md) promoted selectors to first-class declarations: hunts compose selector references, and editing a selector propagates to every referencing hunt on the next refresh.

The current engineering docs still express hunt scope as `(hunt, item)` membership stored in Coverage Memory (C016). Refresh Engine (C008) writes the active set there; Assessment Engine (C009) and Synthesis Engine (C011) read it to enumerate items per hunt. That model duplicates selector semantics: an item's inclusion in a hunt is already implied by the join path item → selector match → hunt references that selector. Storing `(hunt, item)` rows re-materializes a join the architecture already knows how to compute, and it forces Refresh Engine (C008) to walk each hunt's selectors independently even when two hunts share a selector.

Manual items add a second axis. A bird-dogger may assign one manually captured item to several hunts (for example, a cross-team blocker tracked in both `supply-q3` and `supply-leadership`). The current CLI shape — required single `--hunt` on `item add`, reassignment via `item set --hunt` — encodes a sole owning hunt that the multi-hunt use case rejects.

The product requirements already align with silent leave for source-backed items: when an item stops matching a hunt's selectors, it leaves the active set on the next refresh with no separate drop-off surface. That behavior is selector-driven; persisting `(hunt, item)` membership in Coverage Memory (C016) is redundant with selector reconciliation.

Without a recorded decision, downstream doc tasks cannot consistently assign scope edges to Item Store (C007), slim Coverage Memory (C016) to refresh metadata, or document selector-deduped refresh in Refresh Engine (C008).

## Options

- **A. Dual junctions in Item Store (C007).** `item_selectors(item_id, selector_id)` holds source-backed scope materialized on refresh; `item_hunts(item_id, hunt_id)` holds manual multi-hunt assignment written by CLI (C001). Active set for hunt H is computed as a union query over both tables. Coverage Memory (C016) retains only hunt refresh timestamp and per-hunt source-availability snapshot. Cost: scope enumeration moves from a simple C016 read to an Item Store (C007) query; two junction tables to document and implement.

- **B. Unified `item_scope` junction.** One table with `(item_id, scope_kind, scope_ref)` where `scope_kind` is `selector` or `hunt`. Cost: polymorphic foreign keys complicate schema documentation and query patterns; the two scope origins (refresh-materialized vs CLI-materialized) are obscured behind one shape; enforcement that manual rows use `hunt` and refresh rows use `selector` becomes a convention rather than a table boundary.

- **C. Keep `(hunt, item)` in Coverage Memory (C016).** Status quo. Cost: duplicate selector semantics; Refresh Engine (C008) cannot dedupe selector pulls across hunts; multi-hunt manual assignment requires either duplicate membership rows per hunt without a clean manual-only path or continued reliance on a single owning hunt; Assessment Engine (C009) and Synthesis Engine (C011) keep reading membership from the wrong component for source-backed scope.

## Decision

Option A — dual junctions in Item Store (C007), selector-deduped refresh, manual multi-hunt via `item_hunts`.

### Locked decisions

| # | Decision | Choice |
|---|---|---|
| D1 | Materialized scope storage | Dual junctions in Item Store (C007): `item_selectors(item_id, selector_id)` for source-backed scope; `item_hunts(item_id, hunt_id)` for manual multi-hunt assignment |
| D2 | Refresh unit of work | Selector-deduped: collect unique selectors across target hunts; pull each once per bugel; reconcile `item_selectors`; update per-hunt metadata in Coverage Memory (C016) |
| D3 | Manual multi-hunt | `item hunt` sub-commands (`add` / `remove` / `list`); optional repeatable `--hunt` on `item add` seeds initial `item_hunts` rows; no single `owning_hunt` scalar |
| D4 | Active set query | Union: items linked to any of hunt H's selectors ∪ items in `item_hunts` for H |
| D5 | Selector edit mid-quarter | Declaration changes immediately; `item_selectors` reconciles on next full refresh; `--no-refresh` uses last materialized links |
| D6 | Coverage Memory (C016) role | Hunt refresh metadata only: set-level `last_refresh`, per-hunt `(hunt, source)` availability snapshot — no membership rows |

### Canonical active-set definition

```
active_set(H) =
  { item | ∃ selector S : (item, S) ∈ item_selectors ∧ S ∈ hunt(H).selectors }
  ∪
  { item | (item, H) ∈ item_hunts }
```

Where `hunt(H).selectors` is the set of selector names referenced by hunt H, resolved via Selector Registry (C017).

### Schema (documentation-level)

These shapes describe the intended SQLite tables when implementation lands; this ADR does not ship migration SQL.

**`item_selectors(item_id, selector_id)`**

- Primary key or unique constraint on `(item_id, selector_id)`.
- `item_id` references Item Store (C007) internal id (`bdogitem-<n>` per [ADR006](ADR006-native-id-scheme.md)).
- `selector_id` references Selector Registry (C017) selector name (config entity id).
- Optional `matched_at` timestamp for debugging (when the link was last confirmed by a pull).
- Written exclusively by Refresh Engine (C008) during selector reconciliation.
- Source-backed items only: manual items do not receive `item_selectors` rows (enforced by origin tag in Item Store (C007) component contract).

**`item_hunts(item_id, hunt_id)`**

- Primary key or unique constraint on `(item_id, hunt_id)`.
- `item_id` references Item Store (C007) internal id.
- `hunt_id` references Hunt Registry (C006) hunt name.
- Written exclusively by CLI (C001) via `item hunt add` / `item hunt remove` and optionally seeded by repeatable `--hunt` on `item add`.
- Manual items only: source-derived items do not receive `item_hunts` rows.

**Coverage Memory (C016) — slimmed**

- `hunt_refresh(hunt_id, refreshed_at)` — set-level last-refresh marker per hunt.
- `hunt_source_availability(hunt_id, source_id, status, reason, refreshed_at)` — per-hunt source-availability snapshot keyed by `(hunt, source)`.
- No `(hunt, item)` membership table.

### Refresh algorithm (selector-deduped)

Applies during `bdog hunt bugel` when refresh is not skipped (`--no-refresh` absent).

1. **Resolve target hunts.** For a single-hunt bugel, one hunt; for multi-hunt invocations (if supported), the explicit hunt set. Collect all selector names referenced by those hunts via Hunt Registry (C006) → Selector Registry (C017).

2. **Dedupe selectors.** Build the unique set of selector names across all target hunts. Each selector appears once in the pull plan regardless of how many hunts reference it.

3. **Pull per selector.** For each unique selector, resolve to `(source, selector-type, value)` via Selector Registry (C017); invoke Source Adapter (C005) to pull items; upsert item rows and per-item underlying-data-change timestamps in Item Store (C007).

4. **Reconcile `item_selectors` per selector.** For each selector S just pulled:
   - **Add** `(item_id, S)` for every returned item not already linked.
   - **Remove** `(item_id, S)` for every existing link where the item was not in this pull's result set (stale match for S).
   Reconciliation is per-selector, not per-hunt: one selector pull updates scope edges for all hunts that reference S.

5. **Write Coverage Memory (C016) per affected hunt.** For each hunt H in the target set:
   - Update `hunt_refresh(H, refreshed_at)`.
   - Update `hunt_source_availability` rows for each source touched by H's selectors.

6. **Do not write membership to Coverage Memory (C016).** Active set membership is derived from Item Store (C007) junctions at query time.

**`--no-refresh`:** Skip steps 3–5 entirely. Assessment and synthesis run against the last materialized `item_selectors` links and last Coverage Memory (C016) refresh metadata. Selector declaration edits made since the last refresh are visible in config but not reflected in scope until the next full bugel.

**`--dry-run`:** Execute steps 1–3 in preview mode; compute the delta to `item_selectors` and Coverage Memory (C016) metadata without writing. Pointer: CLI (C001) owns the dry-run surface; Refresh Engine (C008) returns the preview delta.

### Edge cases

**Selector edit before refresh.** A bird-dogger edits a selector declaration (`selector set`). The new query text is live in Config Store (C002) and Selector Registry (C017) immediately. `item_selectors` still reflects the last pull until the next full refresh reconciles against the updated query. A bugel with `--no-refresh` uses stale links by design.

**Item matches two selectors in one hunt.** One item, two `(item_id, selector_id)` rows for different selectors in the same hunt. The active-set query dedupes by item id — the item appears once in `active_set(H)`.

**Shared selector across hunts.** One selector S referenced by hunts H1 and H2. Refresh pulls S once; reconciliation updates `item_selectors` for S; both H1 and H2's active sets pick up the change via the shared links. Coverage Memory (C016) receives separate `hunt_refresh` updates for H1 and H2.

**Item out of scope but retained in Item Store (C007).** When an item stops matching a selector, reconciliation removes the `(item_id, selector_id)` row. The item row remains in Item Store (C007) per [ADR007](ADR007-no-cascade-removal.md) — history, notes, overrides, and touch log are preserved. The item simply falls out of `active_set(H)` for hunts that depended on that selector link.

**Manual item in zero hunts.** A manual item may exist with no `item_hunts` rows (all hunts removed via `item hunt remove`). It remains in Item Store (C007) but appears in no hunt's active set until re-assigned.

**Manual item in multiple hunts.** `item hunt add <item> <hunt>...` adds `(item_id, hunt_id)` rows idempotently. Each hunt's active set includes the item via the `item_hunts` branch of the union. Removing from one hunt (`item hunt remove`) does not affect other hunts.

**Hunt removal with manual items.** Per [ADR007](ADR007-no-cascade-removal.md), hunt removal is rejected while manual items still have `item_hunts` rows for that hunt. The bird-dogger clears manual links first (`item hunt remove` for each, or `item remove` to delete the item entirely).

**Source-derived item scope.** Source-derived items enter scope only via `item_selectors` reconciliation. They never receive `item_hunts` rows. Scope adjustment for source-backed items is by editing selectors or hunt selector membership — not by `item hunt` verbs.

### CLI shape for manual capture (D3 detail)

**`item hunt` sub-resource** (mirrors `hunt selector` pattern):

```
bdog item hunt add    <item> <hunt>...
bdog item hunt remove <item> <hunt>...
bdog item hunt list   <item>
```

**`item add`:** `--hunt` becomes optional and repeatable. Each supplied hunt seeds an `item_hunts` row at creation time. Omitting `--hunt` creates a manual item with no hunt membership (the bird-dogger assigns hunts later via `item hunt add`). There is no sole owning hunt.

**`item set --hunt`:** Deprecated in favor of `item hunt add` / `item hunt remove`. Reassignment of manual items across hunts uses the `item hunt` verbs exclusively. See downstream C001 doc updates (task T5).

## Consequences

- **Item Store (C007)** owns materialized scope edges: `item_selectors` (refresh-written) and `item_hunts` (CLI-written). Per-item facts and scope junctions live together; items may exist with zero links (retained history, no current hunt scope).
- **Coverage Memory (C016)** slimmed to Hunt Refresh State: `hunt_refresh` and `hunt_source_availability` only. Prose may say "Hunt Refresh State (C016)" while keeping component id C016 unchanged. No `(hunt, item)` membership.
- **Refresh Engine (C008)** orchestrates selector-deduped pulls and `item_selectors` reconciliation; writes Coverage Memory (C016) refresh metadata per affected hunt; does not write membership rows anywhere.
- **Assessment Engine (C009)** and **Synthesis Engine (C011)** enumerate items per hunt via Item Store (C007) active-set query, not Coverage Memory (C016) membership read. Coverage Memory (C016) may still be read for refresh metadata (freshness display) but not for item enumeration.
- **CLI (C001)** gains `item hunt` sub-commands; `item add` accepts repeatable optional `--hunt`; `item set --hunt` gives way to `item hunt` verbs for manual multi-hunt assignment.
- **Renderer (C015)** displays hunt membership as a list derived from the active-set union (selector overlap ∪ manual links), with origin hint per hunt. See downstream C015 doc updates (task T6).
- **ADR001** consequence "two membership rows in Coverage Memory (C016)" is superseded: shared selector across hunts produces one `item_selectors` row per selector match, visible in every referencing hunt's active set without duplicate membership storage.
- **ADR004** gains a consequence: refresh materializes `item_selectors`; selector edit propagates to scope on the next pull.
- **ADR007** hunt-remove pre-step for manual items becomes: clear all `item_hunts` rows for the hunt (or remove items), not reassign a sole `--hunt`.
- Composes with [ADR001](ADR001-source-hunt-decoupling.md) (source/hunt decoupling unchanged), [ADR004](ADR004-selectors-as-first-class-declarations.md) (selectors remain the scope primitive), and [ADR007](ADR007-no-cascade-removal.md) (no cascade; items retained when scope links drop).
- Future implementation phase (separate from this doc-only ADR): SQLite DDL for junction tables, active-set query helper, Refresh Engine (C008) reconciliation loop, CLI handlers, integration tests for selector edit and multi-hunt manual paths.

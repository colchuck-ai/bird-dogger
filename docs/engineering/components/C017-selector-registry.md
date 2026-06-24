# Selector Registry (C017)

Holds the declared set of selectors and resolves each selector name to an executable `(source, selector-type, value)` triple. Validates that every selector's source exists in Source Registry (C004). Selectors are shared scope primitives: one declaration can back many hunts, and edits propagate to every referencing hunt on the next refresh per [ADR004](../adrs/ADR004-selectors-as-first-class-declarations.md).

## Data model

**Selector declaration.** Each selector is a named row in Config Store (C002) `[[selectors]]` with:

| Field | Required | Notes |
|-------|----------|-------|
| `name` | yes | Bird-dogger-chosen identifier; CLI positional argument |
| `id` | assigned | Stable native id `bdogselector-<n>` per [ADR006](../adrs/ADR006-native-id-scheme.md) |
| `source` | yes | Source **id** (e.g. `bdogsource-1`), not display name |
| `selector-type` | yes | `project`, `jql`, or `filter` |
| `selector` | yes | Query value — project key, JQL string, filter id, etc. |

**Resolved triple.** At refresh or test time, Selector Registry (C017) resolves a selector **name** to:

```
(source_name, selector-type, selector_value)
```

where `source_name` is the registered source's CLI name from Source Registry (C004) (looked up from the stored source id), `selector-type` is the declared type, and `selector_value` is the `selector` field. Source Adapter (C005) consumes this triple to pull items.

**Shared across hunts.** Selector declarations are hunt-independent. Hunt Registry (C006) holds unordered lists of selector **ids** per hunt; many hunts may reference the same selector id. Selector Registry (C017) is the canonical home for the triple; hunts only hold membership references.

### Invariants

- At most one selector row per `name`.
- `source` must reference an existing source id in Source Registry (C004).
- `selector-type` is explicit on every row — never inferred from value shape.
- Selector ids are never reused after removal; counters only increase (Config Store (C002) assignment).
- Cross-hunt sharing is by reference (selector id on hunt rows), not by duplicating triple text.

## Interfaces

Selector Registry (C017) has no CLI surface of its own. CLI (C001) dispatches `selector` verbs; Selector Registry (C017) validates and resolves while Config Store (C002) persists.

### CLI dispatch (reference only)

Full verb shapes live in [CLI (C001)](C001-cli.md#selector). Summary of what crosses into Selector Registry (C017):

| CLI verb | Selector Registry (C017) role |
|----------|-------------------------------|
| `selector add` | Validate source exists; accept triple fields; delegate persist to Config Store (C002) |
| `selector set` | Validate updated source/type/value; delegate persist |
| `selector remove` | Check no hunt references; delegate delete to Config Store (C002) |
| `selector info` / `list` | Read and format declarations |
| `selector test` | Resolve name → triple; invoke Source Adapter (C005) preview pull |

Hunt membership (`hunt selector add|remove|list`) is Hunt Registry (C006) territory; it validates selector names against Selector Registry (C017) but does not mutate selector triples.

### Read

| Caller | Trigger | Selector Registry (C017) action |
|--------|---------|-----------------------------------|
| CLI (C001) | `selector info`, `list`, `test` | Load declarations via Config Store (C002); resolve names |
| Hunt Registry (C006) | Hunt CRUD, `hunt selector` membership | Verify referenced selector ids/names exist |
| Refresh Engine (C008) | `hunt bugel` refresh phase | Resolve each unique selector name in the pull plan to a triple |
| Source Registry (C004) | Source removal pre-check | Enumerate selectors referencing a source id (blocking list for no-cascade) |

Reads use the current Config Store (C002) snapshot for the invocation — no cross-invocation cache.

### Write

| Caller | Trigger | Selector Registry (C017) action |
|--------|---------|-----------------------------------|
| CLI (C001) | `selector add`, `set`, `remove` | Validate source + no-cascade preconditions; mutate via Config Store (C002) |

**Source validation on write.** Every add/set that touches `source` resolves the CLI source name to a source id and confirms Source Registry (C004) recognizes that id before persisting.

**Removal gate (no-cascade).** `selector remove` is rejected when Hunt Registry (C006) reports any hunt still listing the selector id. Error lists blocking hunts and points to `hunt selector remove`. No automatic hunt membership cleanup. See [ADR007](../adrs/ADR007-no-cascade-removal.md).

### Resolution contract

```
resolve(selector_name) → (source_name, selector_type, selector_value) | error
```

- **Not found:** selector name absent from declarations.
- **Dangling source:** `source` id missing from Source Registry (C004) — fail-fast on read paths that need resolution (test, refresh).
- **Rename:** `--rename` on `selector set` updates `name` only; hunt references keyed on id remain valid.

## Behavior

### Declaration lifecycle

1. CLI (C001) receives `selector add` with name, `--source`, `--selector-type`, `--selector`.
2. Selector Registry (C017) resolves `--source` name → source id via Source Registry (C004).
3. Config Store (C002) assigns `bdogselector-<n>` if needed and persists the row.
4. Hunt Registry (C006) may link the selector into hunts via `hunt selector add` (membership only).

Hand-edited TOML follows the same model: bird-dogger omits `id` on new rows; Config Store (C002) assigns on next CLI read. Cross-references in TOML use source **id**; CLI `--source` accepts source **name** for convenience.

### Shared edit propagation

Editing a selector (`selector set` on source, type, or value) is immediate in Config Store (C002) and Selector Registry (C017). Every hunt that references the selector id sees the updated triple on the next resolution — no per-hunt copy to update.

Materialized scope in Item Store (C007) (`item_selectors`) catches up on the next full refresh, not on declaration edit alone. A bugel with `--no-refresh` intentionally uses the last materialized links. See [ADR012](../adrs/ADR012-scope-via-selectors.md) for scope reconciliation; Refresh Engine (C008) owns the pull plan.

### Selector test

`selector test` resolves the triple and asks Source Adapter (C005) for a preview pull (item count or sample). Exercises the query independent of any hunt — the primary affordance called out in ADR004 for first-class selectors.

### Refresh-time resolution (selector-deduped)

Refresh Engine (C008) does not duplicate selector resolution logic inside Selector Registry (C017). Instead:

1. Refresh Engine (C008) collects the **unique** selector names referenced across the target hunt set (via Hunt Registry (C006)).
2. For each unique name, Refresh Engine (C008) calls Selector Registry (C017) `resolve` once.
3. Source Adapter (C005) pulls per resolved triple; `item_selectors` reconciliation and per-hunt metadata writes follow [ADR012](../adrs/ADR012-scope-via-selectors.md).

Selector Registry (C017) supplies triples; it does not dedupe hunts, reconcile junction tables, or write Hunt Refresh State (C016). One selector referenced by three hunts is still resolved once per bugel because Refresh Engine (C008) dedupes **before** calling resolve.

### Removal semantics (no-cascade)

| Step | Action |
|------|--------|
| 1 | Hunt Registry (C006) lists hunts whose `selectors` array includes this selector id |
| 2 | If any exist → fail with hunt names and `hunt selector remove` hints |
| 3 | If clear → Config Store (C002) deletes the selector row |

Removing a selector from a hunt (`hunt selector remove`) does not delete the selector declaration. Source-derived items may lose scope links for that selector on the next refresh reconciliation; item rows remain in Item Store (C007) per ADR007.

Removing a source requires clearing all selectors that reference it first (`selector remove` after hunt unlink), same no-cascade posture.

### Rename semantics

`selector set --rename` updates the `name` field only. Hunt rows reference selector **id**; no hunt membership rewrite. CLI positional arguments use the new name immediately after persist.

## Edge cases

**Selector referenced by multiple hunts.** Valid and expected. One edit updates the triple for all hunts on next resolve/refresh.

**Remove selector still linked to hunts.** Rejected; bird-dogger clears each `hunt selector remove` first (ADR007).

**Remove source with dependent selectors.** Rejected; error lists selectors with `selector remove` hints (after hunt unlink per selector).

**Hand-added selector without id.** Config Store (C002) assigns `bdogselector-<n>` on next CLI read; Selector Registry (C017) treats it like any other declaration once id is present.

**Cross-reference by source name in TOML.** Invalid on read — `source` field must be source id. Use `bdog source info` / `list` to discover ids for hand-editing (Config Store (C002) contract).

**Dangling source id after source removed out-of-band.** Resolution fails on test/refresh with actionable error; declaration row may remain until bird-dogger fixes or removes it.

**Selector edit mid-quarter, bugel with `--no-refresh`.** Declaration shows new query in `selector info`; active set still uses last materialized `item_selectors` until a full refresh (ADR012 D5).

**Same item via two selectors in one hunt.** Item identity is source-scoped in Item Store (C007); two selector matches produce two `item_selectors` rows but one item row. Active-set query dedupes by item id (ADR012).

**Empty selector value or unsupported type.** Rejected at add/set validation before persist.

## Relationships

- **CLI (C001)**: dispatches `selector` CRUD and `selector test`; orchestrates user-visible errors for no-cascade and validation failures.
- **Config Store (C002)**: persistence for `[[selectors]]` rows; id assignment; fresh read every invocation.
- **Source Registry (C004)**: validates `source` id on every write; supplies source name during triple resolution for Source Adapter (C005).
- **Source Adapter (C005)**: executes pulls from resolved triples (`selector test`, refresh).
- **Hunt Registry (C006)**: stores hunt → selector id membership; validates references against Selector Registry (C017); does not own triple text.
- **Refresh Engine (C008)**: collects unique selector names across hunts, resolves via Selector Registry (C017), orchestrates selector-deduped refresh per ADR012.
- **Item Store (C007)**: receives `item_selectors` rows from refresh reconciliation keyed by selector id; does not store selector declarations.

## Success criteria

- Every declared selector resolves to exactly one `(source, selector-type, value)` triple when its source exists in Source Registry (C004).
- `selector add` / `set` reject unknown sources before Config Store (C002) persist.
- `selector remove` fails when any hunt references the selector, with blocking hunt list and clearing verbs (ADR007).
- The same selector id can appear in multiple hunts without duplicating triple text in Config Store (C002).
- `selector test` exercises a query without requiring a hunt declaration.
- Selector declaration edits are visible immediately via `selector info`; scope materialization follows ADR012 on refresh (Refresh Engine (C008) dedupes selector resolution — algorithm not re-specified here).
- Realizes the scope-axis share-at-component-layer posture from ADR004 and supports **J001-O002-R001** (coverage set visibility via `active_set(H)` selector branch) alongside Hunt Registry (C006) and Item Store (C007).

## Notes

### Design decisions

**First-class scope primitive.** Selectors are not embedded in hunt rows. Lifting them to a Connection-band registry (ADR004) lets one JQL back O001 slip detection and O005 leadership hunts without copy-paste drift.

**Triple carries its own source binding.** A hunt may mix selectors across sources because each selector row includes `source`. Cross-source hunts from ADR001 work without special casing.

**Ids for hunt references, names for CLI.** Hunts store selector ids in TOML; bird-doggers use selector names at the shell. Renames are local to the selector row.

**Resolution vs persistence split.** Config Store (C002) owns bytes on disk; Selector Registry (C017) owns validation and the resolve contract consumed by test and refresh paths.

**No-cascade on selector removal.** Silent hunt unlink would hide scope changes; fail-fast keeps teardown visible (ADR007). Source-derived items lose scope via refresh reconciliation, not cascade delete.

### Worked example (shared selector, edit propagates)

```sh
bdog selector add supply-p0 \
    --source ttd-jira \
    --selector-type jql \
    --selector "project = SUPPLY AND priority = P0 AND status != Done"

bdog hunt selector add supply-q3 supply-p0
bdog hunt selector add supply-leadership supply-p0

# Narrow query once — both hunts pick it up on next bugel
bdog selector set supply-p0 \
    --selector "project = SUPPLY AND priority = P0 AND status != Done AND labels != deprecated"
```

### Worked example (no-cascade remove)

```sh
# fails while supply-q3 still references supply-p0
bdog selector remove supply-p0
# Error: selector `supply-p0` is referenced by 2 hunts:
#   - supply-q3         (run: bdog hunt selector remove supply-q3 supply-p0)
#   - supply-leadership (run: bdog hunt selector remove supply-leadership supply-p0)

bdog hunt selector remove supply-q3 supply-p0
bdog hunt selector remove supply-leadership supply-p0
bdog selector remove supply-p0
```

## Open decisions

- **Selector versioning.** ADR004 defers versioning — selectors remain mutable in place until a concrete requirement forces a history model. If hunt wind-down needs frozen query snapshots, that will require a new ADR.
- **Filter-type semantics per source.** `filter` is a valid `selector-type`, but Source Adapter (C005) interpretation per source type (Jira Cloud vs DC vs local-json) is adapter territory — document there when C005 lands.

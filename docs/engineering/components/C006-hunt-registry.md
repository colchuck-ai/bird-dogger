# Hunt Registry (C006)

Holds the declared hunts. Each hunt is a named, unordered set of selector references resolved via Selector Registry (C017), plus an active flag. Hunt Registry (C006) composes scope — it does not hold auth, selector triple text, or item membership. Source credentials live in Source Registry (C004); query text lives in Selector Registry (C017); materialized scope lives in Item Store (C007) per [ADR012](../adrs/ADR012-scope-via-selectors.md).

## Data model

**Hunt declaration.** Each hunt is a named row in Config Store (C002) `[[hunts]]` with:

| Field | Required | Notes |
|-------|----------|-------|
| `name` | yes | Bird-dogger-chosen identifier; CLI positional argument; unit of bugel scoping |
| `id` | assigned | Stable native id `bdoghunt-<n>` per [ADR006](../adrs/ADR006-native-id-scheme.md) |
| `active` | yes | Boolean lifecycle flag; default `true` on CLI `hunt add` |
| `selectors` | yes (may be empty) | Unordered list of selector **ids** |

**Selector membership.** Hunts reference selectors by **id** in TOML. CLI `hunt selector` verbs accept selector **names** for convenience; Hunt Registry (C006) resolves names to ids via Selector Registry (C017) before persist. Order in the `selectors` array has no bird-dogger-visible meaning — refresh walks all referenced selectors without bird-dogger-controlled sequencing per [ADR004](../adrs/ADR004-selectors-as-first-class-declarations.md).

**Active flag.** When `active = true`, the hunt participates in no-arg `hunt bugel` (all active hunts). When `active = false`, the hunt is skipped by default bugel but remains declared: selector membership, refresh history, and manual item links are preserved. Explicit `hunt bugel <name>` or `--all` still targets inactive hunts on demand. End-of-quarter cleanup deactivates rather than removes per CLI (C001) design notes.

### Invariants

- At most one hunt row per `name`.
- Every id in `selectors` must reference an existing selector id in Selector Registry (C017).
- Selector list is a set, not an ordered pipeline — duplicates rejected on `hunt selector add`.
- Hunt ids are never reused after removal; counters only increase (Config Store (C002) assignment).
- Hunts hold membership references only — no inline `(source, selector-type, value)` triple text.

## Interfaces

Hunt Registry (C006) has no CLI surface of its own. CLI (C001) dispatches `hunt` verbs; Hunt Registry (C006) validates and resolves while Config Store (C002) persists.

### CLI dispatch (reference only)

Full verb shapes live in [CLI (C001)](C001-cli.md#hunt). Summary of what crosses into Hunt Registry (C006):

| CLI verb | Hunt Registry (C006) role |
|----------|---------------------------|
| `hunt add` | Create hunt row with `active = true` and empty `selectors` |
| `hunt remove` | Check no-cascade preconditions; delegate delete to Config Store (C002) |
| `hunt info` / `list` | Read and format declarations; `list --inactive` filters on `active` |
| `hunt set --rename` | Update `name` only; delegate persist |
| `hunt activate` / `deactivate` | Flip `active` flag per [ADR009](../adrs/ADR009-side-effecting-actions-on-dedicated-verbs.md) |
| `hunt bugel` | Resolve target hunt set (active filter vs explicit names); hand off to Refresh Engine (C008) — orchestration, not Hunt Registry (C006) persistence |

**hunt selector** (unordered set membership):

| CLI verb | Hunt Registry (C006) role |
|----------|---------------------------|
| `hunt selector add` | Validate each selector name exists in Selector Registry (C017); append ids; reject duplicates |
| `hunt selector remove` | Remove selector ids from hunt row |
| `hunt selector list` | Resolve ids → names for display |

`add` accepts varargs; errors if any selector is already linked; partial state is not written.

### Read

| Caller | Trigger | Hunt Registry (C006) action |
|--------|---------|-----------------------------|
| CLI (C001) | `hunt info`, `list`, `bugel` target resolution | Load declarations via Config Store (C002); filter active/inactive |
| Selector Registry (C017) | `selector remove` pre-check | List hunts whose `selectors` array includes the selector id |
| Refresh Engine (C008) | `hunt bugel` refresh phase | Supply target hunts and their selector name sets for pull-plan construction |
| Assessment Engine (C009), Synthesis Engine (C011), Renderer (C015) | Scoped reads | Hunt name is scope key; membership enumeration goes through Item Store (C007) `active_set(H)`, not Hunt Registry (C006) |

Reads use the current Config Store (C002) snapshot for the invocation — no cross-invocation cache.

### Write

| Caller | Trigger | Hunt Registry (C006) action |
|--------|---------|-----------------------------|
| CLI (C001) | `hunt add`, `set`, `remove`, `activate`, `deactivate` | Validate; mutate via Config Store (C002) |
| CLI (C001) | `hunt selector add`, `remove` | Validate selector names via Selector Registry (C017); mutate membership |

**Selector validation on write.** Every `hunt selector add` and any hand-edited `selectors` list on read resolves each selector id/name through Selector Registry (C017). Unknown selectors fail before persist with actionable error.

**Removal gate (no-cascade).** `hunt remove` is rejected when Item Store (C007) reports manual items still linked via `item_hunts` for this hunt. Error lists blocking items and points to `item hunt remove` or `item remove`. No automatic item unlink. See [ADR007](../adrs/ADR007-no-cascade-removal.md).

### Resolution contract

```
hunt_selectors(hunt_name) → [selector_name, ...] | error
```

- **Not found:** hunt name absent from declarations.
- **Dangling selector id:** id missing from Selector Registry (C017) — fail-fast on read paths that need resolution (info, bugel refresh).
- **Rename:** `--rename` on `hunt set` updates `name` only; selector id references and Hunt Refresh State (C016) keys keyed on hunt name require coordinated updates in downstream components (see Open decisions).

## Behavior

### Declaration lifecycle

1. CLI (C001) receives `hunt add <name>`.
2. Config Store (C002) assigns `bdoghunt-<n>` if needed and persists row with `active = true`, `selectors = []`.
3. Bird-dogger links selectors via `hunt selector add <hunt> <selector>...`.
4. Hunt Registry (C006) validates each selector name against Selector Registry (C017) and appends ids to the hunt row.

Hand-edited TOML follows the same model: bird-dogger omits `id` on new rows; Config Store (C002) assigns on next CLI read. Cross-references in TOML use selector **ids**; CLI positional arguments use selector **names** for convenience.

### Scope composition (not auth)

A hunt's coverage set is the union of:

1. Items linked via `item_selectors` to any selector referenced by the hunt (source-backed scope, refresh-materialized).
2. Items linked via `item_hunts` for the hunt (manual multi-hunt assignment, CLI-materialized).

Hunt Registry (C006) supplies the selector reference set for branch (1). It does not authenticate to external systems — each selector's source binding is resolved when Selector Registry (C017) turns a selector name into a triple for Source Adapter (C005). Cross-source hunts work because selectors carry their own source id per [ADR001](../adrs/ADR001-source-hunt-decoupling.md) and [ADR004](../adrs/ADR004-selectors-as-first-class-declarations.md).

### Active hunt filtering for bugel

| Invocation | Target hunts |
|------------|--------------|
| `bdog hunt bugel` (no args) | All hunts where `active = true` |
| `bdog hunt bugel <hunt>...` | Named hunts regardless of `active` |
| `bdog hunt bugel --all` | Every declared hunt |

Inactive hunts preserve declaration and history without running each cadence. Deactivation is the preferred wind-down path; removal requires clearing manual item links first.

### Selector membership edits

Adding or removing selector references changes which selectors Refresh Engine (C008) considers when building the pull plan for this hunt. Declaration change is immediate in Config Store (C002). Source-backed scope in Item Store (C007) catches up on the next full refresh, not on membership edit alone. A bugel with `--no-refresh` uses last materialized `item_selectors` links per ADR012 D5.

Removing a selector from a hunt (`hunt selector remove`) does not delete the selector declaration. Removing a hunt (`hunt remove`) does not delete referenced selectors.

### Refresh-time handoff

Hunt Registry (C006) does not pull items or reconcile scope. On refresh-enabled bugel:

1. CLI (C001) resolves target hunts via Hunt Registry (C006) active filter or explicit names.
2. Refresh Engine (C008) collects **unique** selector names across those hunts.
3. Selector Registry (C017) resolves each name once; Source Adapter (C005) pulls per triple.
4. Item Store (C007) reconciles `item_selectors`; Hunt Refresh State (C016) receives per-hunt refresh metadata only.

Algorithm detail lives in [ADR012](../adrs/ADR012-scope-via-selectors.md) and Refresh Engine (C008); Hunt Registry (C006) supplies hunt names and selector reference lists.

### Removal semantics (no-cascade)

| Step | Action |
|------|--------|
| 1 | Item Store (C007) lists manual items with `item_hunts` rows for this hunt |
| 2 | If any exist → fail with item ids and `item hunt remove` / `item remove` hints |
| 3 | If clear → Config Store (C002) deletes the hunt row |

Selector Registry (C017) consults Hunt Registry (C006) in the reverse direction: `selector remove` fails while any hunt still lists the selector id.

### Rename semantics

`hunt set --rename` updates the `name` field only. Selector id references on the hunt row are unchanged. Downstream stores keyed by hunt name (Hunt Refresh State (C016), manual `item_hunts`) need a coordinated rename strategy — see Open decisions.

## Edge cases

**Empty selector list.** Valid declaration; hunt refresh pulls nothing until selectors are linked via `hunt selector add`.

**Same selector in multiple hunts.** Valid and expected. One selector declaration backs many hunts; Refresh Engine (C008) dedupes selector pulls across hunts per bugel.

**Duplicate selector on `hunt selector add`.** Rejected; partial state is not written.

**Unknown selector name on add.** Rejected before persist; Hunt Registry (C006) validates via Selector Registry (C017).

**Hand-added hunt without id.** Config Store (C002) assigns `bdoghunt-<n>` on next CLI read; Hunt Registry (C006) treats it like any other declaration once id is present.

**Cross-reference by selector name in TOML.** Invalid on read — `selectors` field must be selector ids. Use `bdog selector info` / `list` to discover ids for hand-editing (Config Store (C002) contract).

**Dangling selector id after selector removed out-of-band.** Resolution fails on bugel/info with actionable error; hunt row may remain until bird-dogger fixes membership.

**Inactive hunt, no-arg bugel.** Skipped. Explicit `hunt bugel <name>` still runs the full cycle for that hunt.

**Deactivate already inactive hunt.** Rejected per ADR009 dedicated-verb semantics (`cannot deactivate an inactive hunt`).

**Remove hunt with manual item links.** Rejected per ADR007; bird-dogger clears `item_hunts` first.

**Selector edit mid-quarter, bugel with `--no-refresh`.** Hunt declaration and selector membership show current config in `hunt info`; active set still uses last materialized scope until full refresh (ADR012 D5).

**Source-backed items when hunt deactivated.** Items remain in Item Store (C007); `item_selectors` links persist until reconciliation on a future refresh that includes those selectors. Deactivation stops default bugel from targeting the hunt; it does not cascade-delete scope edges.

## Relationships

- **CLI (C001)**: dispatches `hunt` CRUD, `hunt selector` membership, and `hunt bugel` target resolution; orchestrates user-visible errors for no-cascade and validation failures.
- **Config Store (C002)**: persistence for `[[hunts]]` rows; id assignment; fresh read every invocation.
- **Selector Registry (C017)**: validates selector references on hunt writes; resolves selector ids to names for display; supplies triple resolution consumed by refresh — Hunt Registry (C006) never duplicates triple text.
- **Source Registry (C004)**: not a direct dependency — auth validation happens when selectors resolve their `source` id during refresh, not at hunt declaration time.
- **Refresh Engine (C008)**: reads target hunts and selector reference sets; selector-deduped pull orchestration per ADR012.
- **Item Store (C007)**: holds materialized scope (`item_selectors`, `item_hunts`); Hunt Registry (C006) does not store `(hunt, item)` membership.
- **Hunt Refresh State (C016)**: per-hunt refresh metadata keyed by hunt name; no membership rows.
- **Assessment Engine (C009), Synthesis Engine (C011), Renderer (C015)**: consume hunt name as scope key; enumerate items via Item Store (C007) `active_set(H)`.

## Success criteria

- Every declared hunt exposes an unordered set of selector ids validated against Selector Registry (C017).
- `hunt selector add` rejects unknown selector names before Config Store (C002) persist.
- `hunt remove` fails when manual items still link to the hunt, with blocking item list and clearing verbs (ADR007).
- No-arg `hunt bugel` runs active hunts only; inactive hunts are skipped unless explicitly named or `--all` is used.
- Hunts compose scope by reference — no auth fields, no inline selector triple text, no `(hunt, item)` membership in Hunt Registry (C006).
- The same selector id can appear in multiple hunts without duplicating selector declarations.
- `hunt deactivate` preserves declaration and history without requiring hunt removal.
- Realizes the scope-axis composition posture from [ADR001](../adrs/ADR001-source-hunt-decoupling.md) and [ADR004](../adrs/ADR004-selectors-as-first-class-declarations.md); supports **J001-O002-R001** (coverage set visibility via `active_set(H)`) alongside Selector Registry (C017) and Item Store (C007).

## Notes

### Design decisions

**Scope composer, not auth holder.** Hunts answer "what coverage set am I chasing?" Auth answers "which external instance can I talk to?" per ADR001. Hunt rows never carry tokens, URLs, or source-type bindings — those live on sources and selectors respectively.

**Unordered selector set.** Bird-dogger does not control pull order across selectors within a hunt. Refresh Engine (C008) dedupes across hunts and walks the unique selector set; ordering is an implementation detail, not a configuration surface.

**Ids in file, names at shell.** Hunts store selector ids in TOML; bird-doggers use selector names in `hunt selector` commands. Selector renames update the selector row only; hunt membership keyed on id stays valid (mirrors Selector Registry (C017) id/name split).

**Active flag vs removal.** Deactivation pauses default cadence without destroying declaration, selector membership, or historical refresh metadata. Removal is explicit teardown after manual item links are cleared.

**Validation vs resolution split.** Hunt Registry (C006) validates that referenced selectors exist. Selector Registry (C017) resolves names to triples at refresh time. Hunt Registry (C006) does not call Source Adapter (C005) directly.

**No-cascade on hunt removal.** Silent unlink of manual items would hide scope changes; fail-fast keeps teardown visible (ADR007).

### Worked example (compose a hunt)

```sh
bdog selector add supply-open \
    --source ttd-jira \
    --selector-type jql \
    --selector "project = SUPPLY AND status != Done AND sprint in openSprints()"

bdog selector add supply-p0 \
    --source ttd-jira \
    --selector-type jql \
    --selector "project = SUPPLY AND priority = P0 AND status != Done"

bdog hunt add supply-q3
bdog hunt selector add supply-q3 supply-open supply-p0

bdog hunt info supply-q3
# selectors: supply-open, supply-p0 (unordered)
# active: true

bdog hunt bugel supply-q3
```

### Worked example (shared selector, inactive wind-down)

```sh
bdog hunt add supply-leadership
bdog hunt selector add supply-leadership supply-p0   # same selector as supply-q3

# End of quarter — pause cadence, keep history
bdog hunt deactivate supply-q3

bdog hunt bugel                    # runs supply-leadership only (active)
bdog hunt bugel supply-q3          # explicit name still runs supply-q3
bdog hunt list --inactive          # shows supply-q3
```

### Worked example (no-cascade remove)

```sh
bdog item add --title "Cross-team blocker" --hunt supply-q3
# → bdogitem-42

# fails while manual item still linked
bdog hunt remove supply-q3
# Error: hunt `supply-q3` has 1 manual item link:
#   - bdogitem-42  (run: bdog item hunt remove bdogitem-42 supply-q3)

bdog item hunt remove bdogitem-42 supply-q3
bdog hunt remove supply-q3
```

## Open decisions

- **Hunt rename propagation.** `hunt set --rename` updates the declaration `name` only. Hunt Refresh State (C016) keys and manual `item_hunts` rows also use hunt name — coordinated rename across stores is not fully specified here; may need component-level follow-up or a dedicated ADR when implementation lands.
- **Multi-hunt bugel selector dedupe edge cases.** ADR012 defines the algorithm at the Refresh Engine (C008) layer; Hunt Registry (C006) supplies reference lists only. Preview/dry-run semantics for overlapping inactive+active hunts are CLI (C001) territory.

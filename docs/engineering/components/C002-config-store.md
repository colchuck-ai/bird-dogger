# Config Store (C002)

Owns the single TOML file for declarative entities (sources, selectors, hunts, contacts, chains). Reads and writes on behalf of CLI (C001) and Connection-band registries; fresh disk read every CLI invocation; no secrets — tokens live in Credential Store (C003) only.

## Data model

**One file, five sibling tables.** All declarative configuration lives in one human-editable TOML file. Each entity kind is a top-level array-of-tables section: `[[sources]]`, `[[selectors]]`, `[[hunts]]`, `[[contacts]]`, `[[chains]]`. The sections are siblings — not nested under one another — so the bird-dogger can edit sources, selectors, and hunts independently by text editor or CLI per [ADR004](../adrs/ADR004-selectors-as-first-class-declarations.md).

**File location (XDG layout).** Per engineering README constraints:

| Platform | Config directory | Config file |
|----------|------------------|-------------|
| Linux | `${XDG_CONFIG_HOME}/bdog/` (default `~/.config/bdog/`) | `config.toml` |
| macOS | OS-equivalent to XDG config home | `config.toml` |
| Windows | OS-equivalent to XDG config home | `config.toml` |

Runtime SQLite data lives separately under `${XDG_DATA_HOME}/bdog/`; Config Store (C002) does not touch the database file.

**Stable ids alongside names.** Every declared row carries:

- **`name`** — bird-dogger-chosen identifier; what CLI positional arguments use.
- **`id`** — auto-assigned native id of shape `bdog<kind>-<n>` where `<kind>` is `source`, `selector`, `hunt`, `contact`, or `chain` and `<n>` is a monotonically increasing integer scoped per kind per [ADR006](../adrs/ADR006-native-id-scheme.md).

Config Store (C002) owns the per-kind integer counters for declarative entities and performs assignment plus write-back when a hand-added row omits `id`.

**Cross-references key on id, not name.** On-disk TOML references between entities use the native id form:

| Referencing entity | Referenced field | Target kind |
|--------------------|------------------|-------------|
| `[[selectors]]` | `source` | source id |
| `[[hunts]]` | `selectors` (unordered list) | selector ids |
| `[[chains]]` | `members` (ordered list) | contact ids |

Item-level owner and chain references live in Item Store (C007) SQLite state, not in Config Store (C002); when those SQLite fields are documented alongside config, they also key on contact/chain ids per ADR006.

**No secret fields.** Source rows carry `type`, `url` or `path`, and stable ids — never a token, password, or keyring pointer. Credential Store (C003) is the sole durable secret store ([ADR010](../adrs/ADR010-tokens-never-via-cli-flag.md)).

### Per-table field shapes

**`[[sources]]`**

| Field | Required | Notes |
|-------|----------|-------|
| `name` | yes | CLI positional |
| `id` | assigned | `bdogsource-<n>` |
| `type` | yes | `jira-cloud`, `jira-dc`, `local-json` |
| `url` | when remote | Base URL for Jira types |
| `path` | when `local-json` | File path for file-backed source |

**`[[selectors]]`**

| Field | Required | Notes |
|-------|----------|-------|
| `name` | yes | CLI positional |
| `id` | assigned | `bdogselector-<n>` |
| `source` | yes | Source **id**, not name |
| `selector-type` | yes | `project`, `jql`, or `filter` |
| `selector` | yes | Query value (JQL, project key, filter id, etc.) |

**`[[hunts]]`**

| Field | Required | Notes |
|-------|----------|-------|
| `name` | yes | CLI positional |
| `id` | assigned | `bdoghunt-<n>` |
| `active` | yes | Boolean; default active on CLI `add` |
| `selectors` | yes (may be empty) | Unordered list of selector **ids** |

**`[[contacts]]`**

| Field | Required | Notes |
|-------|----------|-------|
| `name` | yes | CLI positional |
| `id` | assigned | `bdogcontact-<n>` |
| `email`, `slack`, `jira`, `about` | optional | Channel handles and stable description |

**`[[chains]]`**

| Field | Required | Notes |
|-------|----------|-------|
| `name` | yes | CLI positional |
| `id` | assigned | `bdogchain-<n>` |
| `members` | yes (may be empty) | Ordered list of contact **ids**; position 1 is primary |

### Invariants

- At most one row per `name` within each table.
- Ids are never reused after entity removal; counters only increase.
- Cross-reference targets must exist in the same file (validated on read).
- Tokens and other secrets never appear in any field.

## Interfaces

Config Store (C002) has no CLI surface of its own. CLI (C001) and Connection-band registries call internal read/write contracts.

### Read

| Caller | Trigger | Config Store (C002) action |
|--------|---------|----------------------------|
| CLI (C001) | Every command entry | Load `config.toml` from the XDG config path; parse all five tables into memory |
| Source Registry (C004) | Source resolution | Read `[[sources]]` rows |
| Selector Registry (C017) | Selector resolution | Read `[[selectors]]` rows |
| Hunt Registry (C006) | Hunt resolution | Read `[[hunts]]` rows |
| Contact Registry (C014) | Contact / chain resolution | Read `[[contacts]]` and `[[chains]]` rows |

Reads are always from disk — no in-process cache survives across CLI invocations. Each `bdog` process starts with a fresh parse.

### Write

| Caller | Trigger | Config Store (C002) action |
|--------|---------|----------------------------|
| CLI (C001) | Config CRUD verbs (`source`, `selector`, `hunt`, `contact`, `chain` and sub-resources) | Validate, mutate in-memory model, atomically rewrite `config.toml` |
| CLI (C001) | Hand-edit reconciliation (missing `id` on read) | Assign next `bdog<kind>-<n>`, persist, emit acknowledgement |

**Write atomicity.** Persist the full file on each mutation so partial writes never leave sibling tables inconsistent. Implementation may use write-to-temp-then-rename; the contract is all-or-nothing visible state.

**Id assignment interface.** When a row lacks `id`:

1. Increment the per-kind counter.
2. Set `id = bdog<kind>-<n>`.
3. Rewrite the file.
4. Return the assigned id to CLI (C001) for user-visible acknowledgement.

Config Store (C002) is the canonical id assigner for declarative entities; CLI (C001) orchestrates reconciliation messaging ([CLI (C001)](C001-cli.md#behavior)).

### Example on-disk shape (illustrative)

```toml
[[sources]]
name = "ttd-jira"
id = "bdogsource-1"
type = "jira-cloud"
url = "https://jira.example.com"

[[selectors]]
name = "supply-p0"
id = "bdogselector-1"
source = "bdogsource-1"
selector-type = "jql"
selector = "project = SUPPLY AND priority = P0 AND status != Done"

[[hunts]]
name = "supply-q3"
id = "bdoghunt-1"
active = true
selectors = ["bdogselector-1"]

[[contacts]]
name = "alice"
id = "bdogcontact-1"
email = "alice@example.com"
slack = "alice"

[[chains]]
name = "supply-leadership"
id = "bdogchain-1"
members = ["bdogcontact-1"]
```

Exact key ordering and optional-field omission rules are implementation detail; semantic content above is normative.

## Behavior

### Fresh read every invocation

Config Store (C002) reads `config.toml` from disk at the start of every CLI command. External edits between invocations are visible on the next command without restart — there is no daemon and no watch loop. This matches the CLI-first, one-shot subcommand model ([ADR002](../adrs/ADR002-on-demand-cadence-no-daemon.md)).

### CLI write path

1. CLI (C001) validates verb input and no-cascade preconditions (delegated to registries where applicable).
2. Config Store (C002) loads current file, applies the mutation to the in-memory model, validates cross-references and id uniqueness.
3. Config Store (C002) atomically persists the updated file.
4. Connection-band registries read the updated declarations on subsequent resolution in the same invocation.

Non-secret source fields persist here; token collection and Credential Store (C003) writes happen in the same CLI flow but never touch the TOML file.

### Hand-edit reconciliation

The TOML file is mutable from CLI and text editor per engineering README constraints. Config Store (C002) enforces the hand-edit boundary from [ADR006](../adrs/ADR006-native-id-scheme.md):

**Adding by hand.** Bird-dogger writes `name` and value fields, omits `id`. Next CLI read detects the gap, assigns `bdog<kind>-<n>`, writes back, and CLI emits a one-line acknowledgement (e.g. `assigned bdogselector-7 to selector "supply-incidents"`). Cross-references should be wired afterward — via CLI verbs that resolve names to ids, or by copying the printed id into reference lists.

**Editing by hand.** Renames and field changes are equivalent to CLI `set` / `--rename`. Bird-dogger must not hand-edit or delete existing `id` values.

**Invalid hand-written ids.** An `id` that collides with an existing id or skips the sequence causes fail-fast on the next CLI invocation with instruction to remove the hand-written id and re-run for CLI assignment.

**Cross-references in TOML.** Must use id form. Name resolution is a CLI-argument convenience only; on-disk references that use names are invalid and fail validation on read.

### Rename semantics

`--rename` on declarative entities rewrites only the `name` field. All cross-references keyed on `id` remain valid — no propagation pass. Config Store (C002) persists the name change; registries continue resolving by id internally while CLI arguments accept the new name.

### Removal semantics

Config Store (C002) deletes the row on successful `remove`. No-cascade checks run before delete (CLI/registries reject when references remain). Deleted ids are not recycled.

### Declaration change vs refresh materialization

Edits to selectors, hunts, or sources are live in Config Store (C002) immediately. Source-backed item scope in Item Store (C007) (`item_selectors`) reconciles on the next full refresh via Refresh Engine (C008); a `--no-refresh` bugel intentionally uses stale scope edges until the next pull ([ADR012](../adrs/ADR012-scope-via-selectors.md)).

## Edge cases

- **Hand-added entity without cross-refs:** Valid interim state. Bird-dogger assigns id on first CLI touch, then wires references.
- **Hand-written id collision or sequence skip:** Fail-fast; bird-dogger removes bad id and re-runs so Config Store (C002) assigns a fresh one.
- **Cross-reference by name in TOML:** Rejected on read — references must use ids. Use `bdog <noun> info` / `list` to discover ids for hand-editing.
- **Empty hunt selector list:** Allowed declaration; hunt refresh pulls nothing until selectors are linked.
- **Source rename vs credential key:** Config Store (C002) updates the source `name` field only. Credential Store (C003) keyring entry must track the current source name (implementation detail); bird-dogger may need `source rotate-token` after rename if credential key does not auto-migrate — see [Credential Store (C003)](C003-credential-store.md).
- **Concurrent editors:** Two processes writing the same file without coordination can clobber each other. v1 assumes single-user, single-writer discipline per host; file locking is implementation detail.
- **Missing or corrupt file:** First-run and recovery behavior is CLI territory; Config Store (C002) surfaces parse errors fail-fast with path and line context when possible.

## Relationships

- **CLI (C001):** sole external writer for config CRUD; reads via Config Store (C002) on every command; canonical id-assigner orchestrator for hand-edit acknowledgement text.
- **Credential Store (C003):** no read/write coupling except shared source **name** as lookup key for secrets; never reads or writes Config Store (C002) token fields because none exist.
- **Source Registry (C004):** reads `[[sources]]`; validates declarations against registered types.
- **Selector Registry (C017):** reads `[[selectors]]`; validates source id references against Source Registry (C004).
- **Hunt Registry (C006):** reads `[[hunts]]`; validates selector id references against Selector Registry (C017).
- **Contact Registry (C014):** reads `[[contacts]]` and `[[chains]]`; validates ordered member ids.
- **Item Store (C007):** does not store declarative config; item owner/chain fields in SQLite reference contact/chain ids assigned here.

## Success criteria

- Single TOML file at the XDG config path holds all five declarative tables with no secret fields.
- Every declared row has a stable `bdog<kind>-<n>` id; cross-entity references in TOML use ids exclusively.
- Each CLI invocation re-reads the file from disk; hand edits between invocations appear without restart.
- Hand-added rows without `id` receive assignment and write-back on next CLI read with user-visible acknowledgement.
- Invalid hand-written ids fail fast with actionable recovery guidance.
- `--rename` updates name only; no reference rewrite required.
- Realizes the declaration slice of Source Registration for **J001-O001-R007** and **J001-O002-R007** as part of the shared bundle with Credential Store (C003), Source Registry (C004), and Source Adapter (C005) per [PRD003](../../product/pdrs/PRD003-shared-source-registration.md) and [ADR001](../adrs/ADR001-source-hunt-decoupling.md).
- Selector, hunt, contact, and chain declarations support the scope and escalation surfaces that consume them ([ADR004](../adrs/ADR004-selectors-as-first-class-declarations.md), CLI (C001)).

## Notes

### Design decisions

**Sibling tables, not nesting.** Sources, selectors, and hunts are peers in one file so each axis (auth, scope, people) can be edited independently. Hunts compose scope by referencing selector ids; selectors bind to source ids — matching the two-axis decoupling in ADR001 and the first-class selector model in ADR004.

**Ids for references, names for CLI.** Bird-doggers think in names at the shell; on-disk durability uses ids so renames are local field updates. Config Store (C002) is the persistence layer that makes ADR006's hand-edit rules concrete.

**Fresh read, no cache.** A long-lived daemon would tempt stale config; the one-shot CLI model makes disk the single source of truth every time.

**Secrets explicitly excluded.** One mutable config file for declarations plus OS keyring for tokens keeps the "tokens only through CLI to keyring" constraint auditable — nothing in `config.toml` can leak credentials via backup or version control.

**Per-kind counters, not global.** `bdogsource-7` and `bdoghunt-7` are unrelated; kind prefix is the type disambiguator per ADR006.

### Worked example (hand-add selector, wire hunt)

```toml
# 1. Bird-dogger adds by hand — id omitted
[[selectors]]
name = "supply-incidents"
source = "bdogsource-1"
selector-type = "jql"
selector = "project = SUPPLY AND issuetype = Incident AND status != Done"
```

```sh
# 2. Next CLI touch assigns id and writes back
bdog selector info supply-incidents
# assigned bdogselector-4 to selector "supply-incidents"
# (acknowledgement on first read that performed assignment)

# 3. Wire hunt membership via CLI (resolves names → ids in file)
bdog hunt selector add supply-q3 supply-incidents
```

Alternatively, after step 2, edit `[[hunts]]` by hand to append `"bdogselector-4"` to the hunt's `selectors` list.

## Open decisions

- **TOML library selection:** Engineering README commits to TOML for human-editable config; the specific Go parser/encoder (strictness, comment preservation on round-trip, key ordering) is deferred ADR territory — same posture as SQLite driver and keyring library choices in the README Technology Choices section.

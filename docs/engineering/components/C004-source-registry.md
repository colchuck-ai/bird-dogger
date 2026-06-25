# Source Registry (C004)

Holds the declared set of sources and resolves each source name to a `(declaration, secret, adapter)` bundle on demand. Validates source declarations on edit and on read. Owns the registry layer of Source Registration — declarations live in Config Store (C002), secrets in Credential Store (C003), wire behavior in Source Adapter (C005) per [ADR001](../adrs/ADR001-source-hunt-decoupling.md) and [PRD003](../../product/pdrs/PRD003-shared-source-registration.md).

## Data model

**Source declaration.** Each source is a named row in Config Store (C002) `[[sources]]` with:

| Field | Required | Notes |
|-------|----------|-------|
| `name` | yes | Bird-dogger-chosen identifier; CLI positional argument |
| `id` | assigned | Stable native id `bdogsource-<n>` per [ADR006](../adrs/ADR006-native-id-scheme.md) |
| `type` | yes | `jira-cloud`, `jira-dc`, or `local-json` — fixed at `add` |
| `url` | when remote | Base URL for Jira types |
| `path` | when `local-json` | File path for file-backed source per [ADR005](../adrs/ADR005-local-json-source-type.md) |

**Registered source types (v1).**

| Type | Endpoint field | Carries secret | Credential Store (C003) |
|------|----------------|----------------|-------------------------|
| `jira-cloud` | `url` | yes | keyring / env on add, rotate, read |
| `jira-dc` | `url` | yes | same as cloud |
| `local-json` | `path` | no | not consulted |

Type is immutable after `add`. `set` may change `name` (`--rename`), `url`, or `path` — never `--type`.

**Resolved bundle.** At resolution time, Source Registry (C004) assembles:

```
(declaration, secret, adapter)
```

| Leg | Owner | Content |
|-----|-------|---------|
| `declaration` | Config Store (C002) via Source Registry (C004) | The `[[sources]]` row: name, id, type, url/path |
| `secret` | Credential Store (C003) | Opaque token string for token-bearing types; absent for `local-json` |
| `adapter` | Source Adapter (C005) | Type-specific adapter instance bound to this source name and declaration |

The bundle is the unit Refresh Engine (C008), `source test`, and Selector Registry (C017) triple resolution consume when they need a live connection to one external instance.

**Lookup keys.**

- CLI and bird-dogger-facing resolution use source **name**.
- Cross-entity references in TOML (e.g. selector `source` field) use source **id**.
- Source Registry (C004) maintains both views: name → row, id → row.

### Invariants

- At most one source row per `name`.
- Source ids are never reused after removal; counters only increase (Config Store (C002) assignment).
- No secret fields on the declaration — tokens never appear in TOML.
- Every declared `type` must be a registered source type Source Adapter (C005) can instantiate.
- Token-bearing types must have a resolvable secret before authenticated adapter calls; missing secret is a resolution failure, not a silent empty bundle.

## Interfaces

Source Registry (C004) has no CLI surface of its own. CLI (C001) dispatches `source` verbs; Source Registry (C004) validates and resolves while Config Store (C002) and Credential Store (C003) persist their respective legs.

### CLI dispatch (reference only)

Full verb shapes live in [CLI (C001)](C001-cli.md#source). Summary of what crosses into Source Registry (C004):

| CLI verb | Source Registry (C004) role |
|----------|-----------------------------|
| `source add` | Validate type + endpoint fields; delegate declaration persist; orchestrate credential write for token-bearing types |
| `source set` | Validate type-appropriate field edits; reject `--url` on `local-json` and `--path` on remote types |
| `source remove` | Check no-cascade preconditions; delegate declaration delete and credential delete |
| `source info` / `list` | Read and format declarations (no secret values in output) |
| `source test` | Resolve bundle; invoke Source Adapter (C005) connectivity / parse check |
| `source rotate-token` | Confirm type carries secret; delegate credential replace — declaration unchanged |

### Read

| Caller | Trigger | Source Registry (C004) action |
|--------|---------|-------------------------------|
| CLI (C001) | `source info`, `list`, `test` | Load declarations via Config Store (C002); resolve name → bundle when needed |
| Selector Registry (C017) | `selector add/set`, triple resolution | Resolve source name or id → declaration; fail if absent |
| Refresh Engine (C008) | `hunt bugel` pull plan | Supply adapter instances per unique source in the selector set |
| Source Adapter (C005) | Per-request auth | Receive bound source context from registry resolution (not direct Config Store reads for routine paths) |

Reads use the current Config Store (C002) snapshot for the invocation — no cross-invocation cache.

### Write

| Caller | Trigger | Source Registry (C004) action |
|--------|---------|-----------------------------------|
| CLI (C001) | `source add`, `set`, `remove` | Validate declaration shape and type rules; mutate via Config Store (C002) |
| CLI (C001) | `source add`, `rotate-token` | After declaration validation, delegate secret write to Credential Store (C003) |
| CLI (C001) | `source remove` (token-bearing) | Delegate credential delete alongside declaration delete |

**Validation on write.** Every add/set validates:

1. **Type-endpoint consistency** — remote types require `url`; `local-json` requires `path`; cross-field assignments rejected.
2. **Immutability** — `type` cannot change after `add`.
3. **Name uniqueness** — `add` errors on duplicate name; `set --rename` errors on collision.
4. **Registered type** — unknown `--type` values fail before persist.

**Validation on read.** Every path that resolves a source for side effects (test, refresh, selector resolution) re-validates:

1. Declaration row exists for the requested name or id.
2. Required endpoint field present for the declared type.
3. For token-bearing types, Credential Store (C003) can resolve a secret (or surfaces miss to caller).
4. Source Adapter (C005) can construct an adapter for the type.

Hand-edited TOML with invalid type/field combinations fails on the next CLI read with actionable error — same posture as Config Store (C002) cross-reference validation.

**Removal gate (no-cascade).** `source remove` is rejected when Selector Registry (C017) reports any selector still referencing the source id. Error lists blocking selectors and points to `selector remove`. No automatic selector cleanup. See [ADR007](../adrs/ADR007-no-cascade-removal.md).

### Resolution contract

```
resolve(source_name) → (declaration, secret, adapter) | error
resolve_by_id(source_id) → declaration | error
```

- **Not found:** source name or id absent from declarations.
- **Invalid declaration:** type/endpoint mismatch or missing required field — fail-fast before adapter construction.
- **Secret miss:** token-bearing type with no keyring entry and no env fallback — propagate to caller; Source Adapter (C005) surfaces auth failure via availability, not silent success.
- **Rename:** `--rename` on `source set` updates `name` only; selector references keyed on id remain valid; credential key must track current source name (see [Credential Store (C003)](C003-credential-store.md)).

For `local-json`, `secret` is intentionally absent; the bundle is `(declaration, ∅, adapter)`.

## Behavior

### Declaration lifecycle

1. CLI (C001) receives `source add <name>` with `--type` and type-appropriate endpoint flag.
2. Source Registry (C004) validates the declaration shape and registered type.
3. Config Store (C002) assigns `bdogsource-<n>` if needed and persists the row.
4. For token-bearing types, CLI (C001) collects the secret (prompt or env) and Credential Store (C003) writes the keyring entry — Source Registry (C004) does not store the secret.
5. `local-json` skips step 4 entirely per [ADR005](../adrs/ADR005-local-json-source-type.md).

Hand-edited TOML follows the same model: bird-dogger omits `id` on new rows; Config Store (C002) assigns on next CLI read. Selectors reference source **id** in the file; CLI `--source` accepts source **name** for convenience.

### Bundle resolution

When a caller needs a live source connection:

1. Source Registry (C004) loads the declaration from Config Store (C002) by name or id.
2. Re-validates declaration fields for the declared type (read-time check catches hand-edit drift).
3. For token-bearing types, requests the opaque secret from Credential Store (C003) scoped to the source name.
4. Constructs or retrieves a Source Adapter (C005) instance bound to `(name, declaration, secret?)`.
5. Returns the `(declaration, secret, adapter)` tuple to the caller.

Source Registry (C004) does not perform HTTP or file I/O itself — the adapter leg owns wire behavior. Source Registry (C004) owns **which** adapter binds to **which** registered source and that the declaration + secret legs are consistent before handoff.

### `source test`

1. Resolve full bundle for the named source.
2. Source Adapter (C005) runs type-specific probe: authenticated ping for Jira types; file exists / parses / emits well-shaped items for `local-json`.
3. Result surfaces to CLI — same verb, different adapter behavior per type per ADR005.

### Auth axis vs scope axis

Source Registry (C004) answers "who can we talk to?" — one registration per external instance. Hunt Registry (C006) and Selector Registry (C017) answer "what queries define coverage?" — selectors carry source ids; hunts compose selector ids. A single source backs many selectors; selectors back many hunts. Source Registry (C004) is not involved in hunt membership or selector triple text beyond validating that referenced source ids exist when selectors write.

Cross-source hunts work because each selector row carries its own `source` id; bundle resolution happens per unique source at refresh time, not per hunt declaration.

### Rename and endpoint edits

- **`--rename`:** Updates declaration `name` only. Selector `source` ids unchanged. Credential Store (C003) entry must track the new name (implementation detail); bird-dogger may need `source rotate-token` after rename if key does not auto-migrate.
- **`--url` / `--path` set:** Updates endpoint field only; type and id unchanged. Next `source test` or refresh uses the new endpoint; credentials unchanged for Jira types.

## Edge cases

- **Tokenless type on add:** `local-json` never creates a Credential Store (C003) slot; bundle resolution skips secret leg.
- **Secret miss at refresh:** Bundle resolution fails the secret leg; Source Adapter (C005) reports source unavailable per [J001-O001-R006](../../product/outcomes/J001-O001-slip-detection-lag/requirements/J001-O001-R006-source-availability.md) — items are not silently treated as healthy.
- **Hand-edited type/field mismatch:** e.g. `jira-cloud` row with `path` instead of `url` — fail on next read with field guidance.
- **Dangling source id in selector row:** Selector Registry (C017) fails on resolution; Source Registry (C004) confirms id absent when enumerating blockers.
- **`set --url` on local-json / `set --path` on remote:** Rejected at validation — type-endpoint pairing is fixed.
- **Remove with referencing selectors:** No-cascade rejection lists selectors and recovery verbs.
- **Concurrent editors:** Same as Config Store (C002) — v1 assumes single-user discipline; Source Registry (C004) does not add cross-process locking.
- **Unknown future type in hand-edited file:** Fail on read until CLI `source add` uses a registered type enum value.

## Relationships

- **CLI (C001):** dispatches `source` verbs; orchestrates declaration validation, credential collection, and bundle resolution messaging. Verb catalog lives in C001 — not duplicated here.
- **Config Store (C002):** persists `[[sources]]` rows; Source Registry (C004) reads declarations on every resolution path.
- **Credential Store (C003):** supplies the secret leg for token-bearing types on demand; no TOML coupling. Removal coupled to `source remove`.
- **Source Adapter (C005):** provides the adapter leg; instantiated against a registered source; reads secrets only via Credential Store (C003), never from CLI flags.
- **Selector Registry (C017):** validates `source` id on selector write; resolves source name during triple assembly; consults Source Registry (C004) for removal pre-checks.
- **Hunt Registry (C006):** no direct dependency — hunts reference selectors, not sources. Auth for a hunt's refresh is indirect: hunt → selector ids → source ids → Source Registry (C004) bundles.
- **Refresh Engine (C008):** consumes adapter instances from bundle resolution when executing the pull plan.
- **Item Store (C007), Hunt Refresh State (C016):** no direct read/write — source-scoped item identity uses adapter-emitted source ids, not Source Registry (C004) state.

## Success criteria

- Every declared source resolves to exactly one `(declaration, secret, adapter)` bundle when its type and endpoint are valid and secrets exist where required.
- Validation runs on both write (`add`, `set`) and read (test, refresh, selector resolution) paths.
- Doc states the three bundle legs and which component owns each — no secret fields on declarations.
- Token-bearing vs tokenless types follow ADR005 rules; `local-json` bundle has no secret leg.
- `source remove` fails with actionable no-cascade errors when selectors still reference the source id.
- Relationships to Config Store (C002), Credential Store (C003), Source Adapter (C005), and Hunt Registry (C006) (indirect via selectors) are explicit.
- Realizes the registry slice of Source Registration for **J001-O001-R007** and **J001-O002-R007** as part of the shared bundle with Config Store (C002), Credential Store (C003), and Source Adapter (C005) per PRD003 and ADR001.
- Does not duplicate the C001 `source` verb surface — references CLI (C001) for verb shapes only.

## Notes

### Design decisions

**Registry, not store.** Source Registry (C004) is the Connection-band index over Config Store (C002) declarations plus runtime binding to Credential Store (C003) and Source Adapter (C005). It does not introduce a second persistence format — the TOML row is the declaration of record.

**Bundle as the handoff unit.** Callers that need to talk to an external instance ask for one bundle, not three separate lookups. That keeps auth-axis wiring in one place and lets Source Adapter (C005) stay source-type-agnostic at the contract layer per ADR001.

**Validation on read and write.** Write validation catches CLI mistakes early. Read validation catches hand-edited TOML drift and partial deletes without requiring a separate config linter.

**Names at the shell, ids in the file.** Source Registry (C004) mirrors the ADR006 pattern used across declarative entities: bird-doggers use names in CLI positionals; selectors and other references persist source ids.

**Hunts do not reference sources directly.** The engineering README's auth/scope decoupling lands as: Source Registry (C004) for connections, Selector Registry (C017) for query triples, Hunt Registry (C006) for selector membership. C006's indirect relationship is intentional — changing a hunt's sources means editing selector declarations or membership, not hunt rows.

### Worked example (register, test, wire selector)

```sh
bdog source add ttd-jira --type jira-cloud --url https://jira.example.com
# declaration persisted; token written to keyring

bdog source test ttd-jira
# bundle resolved → Jira adapter probes auth + connectivity

bdog selector add supply-open \
    --source ttd-jira \
    --selector-type jql \
    --selector "project = SUPPLY AND status != Done"
# Selector Registry (C017) resolves ttd-jira → bdogsource-1 via Source Registry (C004)
```

```sh
bdog source add fixtures --type local-json --path ~/bdog/items.json
bdog source test fixtures
# bundle: (declaration, no secret, local-json adapter)
```

## Open decisions

- **Adapter instance lifecycle.** Whether Source Registry (C004) constructs a fresh Source Adapter (C005) per CLI invocation or caches instances within one process is implementation detail — the contract is resolve-on-demand with no cross-invocation cache of declarations. Adapter pooling policy is deferred to Source Adapter (C005) component doc / ADR territory.
- **Credential key migration on rename.** Credential Store (C003) notes that the keyring entry should track the current source name; automatic migration vs require `rotate-token` after `--rename` is not yet recorded in an ADR.
- **Future source types.** Additional types (trackers, doc systems, message platforms) register the same way: declare type, endpoint shape, and whether Credential Store (C003) participates. Per-type selector-type vocabularies stay with Source Adapter (C005); Source Registry (C004) only validates that the type enum is known and endpoint fields match.

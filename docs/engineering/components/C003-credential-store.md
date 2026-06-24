# Credential Store (C003)

Owns source-token storage and retrieval via OS keyring with `BDOG_SOURCE_<NAME>_TOKEN` env-var fallback. Returns opaque secret strings only — auth header shaping is Source Adapter (C005) territory. Never persists secrets to Config Store (C002) or disk in plaintext.

## Data model

OS keyring-backed (macOS Keychain, Linux Secret Service, Windows Credential Manager); env-var fallback for headless read and write. No plaintext disk storage.

**One secret slot per source name.** Credential Store (C003) keys credentials by the bird-dogger-chosen source name (the same name used in `bdog source add <name>` and Config Store (C002) declarations). There is no separate credential id; the source name is the lookup key.

**Stored value.** An opaque secret string — typically an API token or password — with no type metadata at the Credential Store (C003) layer. Source type (`jira-cloud`, `jira-dc`, …) determines whether a slot exists at all; Credential Store (C003) does not branch on auth scheme.

**Persistence shape.**

| Layer | Key | Value | When used |
|-------|-----|-------|-----------|
| OS keyring | Source-scoped entry (service/account naming is implementation detail) | Secret string | Primary read/write path |
| Environment | `BDOG_SOURCE_<NAME>_TOKEN` | Secret string | Read fallback when keyring miss or headless write path |

**Env-var naming (load-bearing).** `<NAME>` is the source name uppercased with `-` replaced by `_`. Example: source `ttd-jira` → `BDOG_SOURCE_TTD_JIRA_TOKEN`. The same rule applies on read and on the headless write path.

**No TOML field.** Config Store (C002) source declarations carry `type`, `url`/`path`, and stable ids — never a token field. Credential Store (C003) is the sole durable secret store.

**Tokenless sources.** Source types that declare no secret (e.g. `local-json` per [ADR005](../adrs/ADR005-local-json-source-type.md)) have no Credential Store (C003) slot. Credential Store (C003) is not consulted on add, rotate, read, or remove for those types.

### Invariants

- At most one active secret per source name.
- Credential removal is coupled to source removal — there is no "declaration without credential" partial state ([ADR010](../adrs/ADR010-tokens-never-via-cli-flag.md)).
- Credential Store (C003) never persists flag-delivered secrets; write inputs are interactive terminal reads or env-var reads only.

## Interfaces

Credential Store (C003) has no CLI surface of its own. CLI (C001) and Source Adapter (C005) call internal contracts.

### Write

| Caller | Trigger | Input mechanism | Credential Store (C003) action |
|--------|---------|-----------------|------------------------------|
| CLI (C001) | `bdog source add <name>` when the source type carries a secret | Interactive prompt when stdin is a TTY; else read `BDOG_SOURCE_<NAME>_TOKEN` | Create keyring entry for `<name>` |
| CLI (C001) | `bdog source rotate-token <name>` | Same as add | Replace keyring entry for `<name>` |

**Foreclosed write shapes.** No API accepts a secret string originating from a CLI flag value (`--token`, `--clear-token`, or equivalent). Credential Store (C003) must reject or never expose an interface that accepts flag-sourced secrets — the constraint is architectural per [ADR010](../adrs/ADR010-tokens-never-via-cli-flag.md).

**No other write paths.** Config edits (`source set`), selector/hunt/contact CRUD, bugel, and adapter refresh do not write Credential Store (C003). Token rotation is exclusively `source rotate-token`, not a flag on `source set` ([ADR009](../adrs/ADR009-side-effecting-actions-on-dedicated-verbs.md)).

### Read

| Caller | Trigger | Scope | Credential Store (C003) action |
|--------|---------|-------|--------------------------------|
| Source Adapter (C005) | Per outbound request for a token-bearing source | Single `<source_name>` | Resolve secret: keyring first, then `BDOG_SOURCE_<NAME>_TOKEN` |

Reads are **scoped per source**. Source Adapter (C005) requests the credential for the bound source only; Credential Store (C003) does not bulk-export or list secrets.

**Read fallback order.**

1. OS keyring entry for the source name.
2. If absent or unreadable, `BDOG_SOURCE_<NAME>_TOKEN` in the process environment.
3. If still absent, propagate miss to caller (adapter surfaces auth failure via source availability).

### Delete

| Caller | Trigger | Credential Store (C003) action |
|--------|---------|--------------------------------|
| CLI (C001) | `bdog source remove <name>` for a token-bearing source | Delete keyring entry for `<name>` |

There is no standalone "clear token" operation. Removing a credential without removing the source declaration is foreclosed ([ADR010](../adrs/ADR010-tokens-never-via-cli-flag.md)).

## Behavior

### On `source add` (token-bearing types)

1. CLI (C001) validates declaration fields and persists the non-secret declaration to Config Store (C002).
2. For types that declare a secret, CLI (C001) collects the secret via interactive prompt or env-var — never via flags.
3. Credential Store (C003) writes the secret to the OS keyring under the new source name.
4. Tokenless types skip steps 2–3 entirely ([ADR005](../adrs/ADR005-local-json-source-type.md)).

### On `source rotate-token`

1. CLI (C001) confirms the source exists and its type carries a secret.
2. CLI (C001) collects the replacement secret (prompt or env-var).
3. Credential Store (C003) replaces the keyring entry atomically for that source name.
4. Config Store (C002) is unchanged — rotation is credential-only.

### On adapter request (read path)

When Source Adapter (C005) prepares an authenticated request:

1. Credential Store (C003) resolves the secret for the bound source name.
2. Source Adapter (C005) applies the secret using type-specific auth mechanics (header shape, Basic encoding, etc.).
3. Credential Store (C003) does not participate in HTTP wiring — it returns the string only.

### On `source remove`

When CLI (C001) removes a token-bearing source (after no-cascade checks pass), Credential Store (C003) deletes the keyring entry alongside Config Store (C002) declaration removal. Env-var fallbacks are process-local and are not "removed" by Credential Store (C003).

### Headless vs interactive

| Environment | Write behavior | Read behavior |
|-------------|----------------|---------------|
| TTY | Interactive prompt on add/rotate | Keyring, then env |
| Non-TTY (CI, scripts) | Requires `BDOG_SOURCE_<NAME>_TOKEN` set for write | Keyring, then env |

CLI help for `source add` and `source rotate-token` documents both paths ([ADR010](../adrs/ADR010-tokens-never-via-cli-flag.md)).

## Edge cases

- **Tokenless type on add:** `local-json` and future types with no secret never call write or read. No empty keyring slot is created.
- **Keyring miss, env present:** Read succeeds from env even when keyring entry is absent — supports CI injection without persisting to keyring.
- **Keyring hit, env also set:** Keyring wins on read; env is fallback only.
- **Rotate with wrong env name:** Env-var name derives from source name, not display label. Renaming a source (`source set --rename`) updates Config Store (C002) only; the keyring entry must move with the rename (implementation detail) or the bird-dogger must rotate after rename — component doc expectation: credential key tracks current source name.
- **Missing secret at refresh:** Source Adapter (C005) receives a miss; availability reports auth failure; items are not silently treated as healthy ([J001-O001-R006](../../product/outcomes/J001-O001-slip-detection-lag/requirements/J001-O001-R006-source-availability.md)).
- **No `--clear-token`:** A source without a valid credential is not a supported partial state. Fix via `source rotate-token` or remove the source entirely.
- **Flag-shaped future verbs:** Any future CLI surface that accepted `--token <value>` would violate Credential Store (C003)'s write contract regardless of verb name ([ADR010](../adrs/ADR010-tokens-never-via-cli-flag.md)).
- **Secrets never in process argv:** Interactive and env-var paths keep secrets out of shell history and `ps` argument vectors; Credential Store (C003) write APIs must not encourage flag delivery.

## Relationships

- **CLI (C001)**: sole writer via `source add` and `source rotate-token`; deleter via `source remove`. Collects secrets from prompt/env and delegates persistence to Credential Store (C003). Never writes secrets to Config Store (C002).
- **Config Store (C002)**: holds source declarations only. No secret fields; no read/write coupling except shared source name as lookup key.
- **Source Registry (C004)**: resolves `(declaration, secret, adapter)` bundles; secret leg comes from Credential Store (C003) on demand.
- **Source Adapter (C005)**: sole routine reader — per-request, scoped to one source. Receives opaque secret string; owns auth method and wire format. Must not accept tokens from CLI or flags directly ([ADR010](../adrs/ADR010-tokens-never-via-cli-flag.md)).
- **Item Store (C007), Hunt Refresh State (C016), Renderer (C015)**: no interaction — Credential Store (C003) does not read or write chase-state or rendering data.

## Success criteria

- Doc states keyring-first resolution with `BDOG_SOURCE_<NAME>_TOKEN` fallback, including the name→env transformation rule.
- Write paths are exactly `source add` (new token) and `source rotate-token` (replacement), both via interactive prompt or env-var — never CLI flags.
- Doc explicitly forecloses `--token`, `--clear-token`, and flag-equivalent write paths per [ADR010](../adrs/ADR010-tokens-never-via-cli-flag.md).
- Tokenless source types skip Credential Store (C003) entirely per [ADR005](../adrs/ADR005-local-json-source-type.md).
- Read path is scoped per source for Source Adapter (C005); auth scheme is adapter concern, not Credential Store (C003).
- Secrets are never stored in Config Store (C002) or documented as TOML fields.

## Notes

### Design decisions

**Keyring-first, env-second.** The OS keyring (macOS Keychain, Linux Secret Service, Windows Credential Manager) is the durable, interactive-workstation default. Env-vars cover CI and scripted headless runs without writing secrets to shell history via flags ([ADR010](../adrs/ADR010-tokens-never-via-cli-flag.md)).

**Type-agnostic storage.** Credential Store (C003) stores one string per source name. Whether the source is `jira-cloud`, `jira-dc`, or a future token-bearing type is irrelevant at this layer — the source type's "carries a secret?" declaration gates whether Credential Store (C003) is invoked ([ADR005](../adrs/ADR005-local-json-source-type.md)).

**Auth method is Source Adapter (C005)'s concern.** Jira adapters may map the same PAT string to Bearer headers; a future OAuth type might store a refresh token string. Credential Store (C003) does not interpret or transform the secret.

**Write-path minimization.** Only two verbs mutate credentials. Side-effecting credential operations stay off `source set` so idempotency and audit semantics stay clear ([ADR009](../adrs/ADR009-side-effecting-actions-on-dedicated-verbs.md)). Credential removal is bundled with `source remove`, not a separate clear-token verb.

**Single registration bundle.** Config Store (C002) declaration + Credential Store (C003) secret + Source Registry (C004) + Source Adapter (C005) realize one Source Registration capability ([PRD003](../../product/pdrs/PRD003-shared-source-registration.md), [ADR001](../adrs/ADR001-source-hunt-decoupling.md)).

### Worked example (headless CI)

```sh
export BDOG_SOURCE_TTD_JIRA_TOKEN="***"
bdog source add ttd-jira --type jira-cloud --url https://jira.example.com
# no prompt; token written to keyring from env
bdog source test ttd-jira
```

Interactive workstation equivalent prompts at add time; the keyring entry persists across shells without keeping the env-var set.

## Open decisions

- **Keyring library selection.** Engineering README commits to OS-native keyring storage; the specific Go library (service name, account label schema, error mapping) is deferred ADR territory. Until recorded, implementers should prefer cross-platform keyring abstractions that map to Keychain / Secret Service / Credential Manager without plaintext disk fallback.

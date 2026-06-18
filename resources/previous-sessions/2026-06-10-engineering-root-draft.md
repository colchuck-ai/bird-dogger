# 2026-06-10 — Engineering Root Draft

Working doc for advancing `docs/engineering/`. Outcome of this
session: a first draft of `docs/engineering/README.md` (the
architecture root) for the `birddog` CLI tool, and the cleanup of
stale session-number references inside `docs/product/pdrs/` (those
artifacts no longer exist).

## Purpose

Author the engineering root document from the established product
intent (one job, five outcomes, 31 requirements, four PDRs).
"High-level and concise" was the user's framing — components named
and mapped, but minutia held for the per-component pass.

## Outcome

- Created `docs/engineering/README.md` (architecture root).
- 4 bands (Edge, Connection, State, Reasoning), 15 components
  (C001–C015), all 31 requirements mapped.
- All four product PDRs carried forward as engineering principles.

## Inputs Loaded

For continuity if a future session needs to rebuild context:

- `docs/product/README.md`
- All 5 outcome READMEs under `docs/product/outcomes/`.
- All 31 requirement docs (R001–Rmax under each outcome).
- All 4 PDRs under `docs/product/pdrs/`.
- `intent` skill (`SKILL.md`) including the architecture template
  and the `references/` files for context tracing, structure, and
  workflows.

## Decisions Locked This Session

| Decision | Choice | Why |
| --- | --- | --- |
| Runtime / packaging | Single Go static binary, named `birddog` (avoids `bd` collision with `beads`). | User constraint; matches CLI-first / local-only fit. |
| State storage | One SQLite database under `${XDG_DATA_HOME}/birddog/`. | Indices for ranking and freshness queries; ACID; single-file footprint. |
| Configuration | One TOML file under `${XDG_CONFIG_HOME}/birddog/`. CLI commands and manual edits both valid; tokens never in TOML. | Human-editable, less ambiguous than YAML, supports both bird-dogger and version-control workflows. |
| Secrets | OS keyring (Keychain / Secret Service / Credential Manager); env-var fallback `BIRDDOG_SOURCE_<NAME>_TOKEN` for CI/headless. | Keeps tokens off disk in plaintext; one auth seam regardless of source type. |
| CLI shape | One-shot subcommands only in v1. List-level rendering is tabular text. | Simpler, scriptable, composable. TUI revisit deferred to ADR. |
| Refresh cadence | On-demand inside `birddog checkpoint` (or explicit `birddog refresh`). No daemon. | Covers all the "before each checkpoint" requirements without the daemon surface area. |
| Source access | Read-only against external sources. | Nothing in the requirements asks for writes; overrides/triage/touch logs are local. |
| Escalation send | Out-of-tool. The bird-dogger sends; the tool records under O005-R005. | Keeps notification subsystem out of the architecture entirely. |
| Source vs hunt | Decoupled: a Source authenticates a connection; a Hunt scopes coverage via `(source, selector)` entries. | Lets one hunt mix multiple Jira instances with mixed selectors (project, JQL). |
| First source type | Jira (Cloud first; DC patterns inform the abstraction). Source Adapter contract is source-type-agnostic. | User direction. |

## Architecture Bands and Components

For quick recall — full text is in the README:

- **Edge**: C001 CLI, C002 Config Store, C003 Credential Store, C015 Renderer.
- **Connection**: C004 Source Registry, C005 Source Adapter, C006 Hunt Registry.
- **State**: C007 Item Store, C010 Override Store, C012 Triage Memory, C013 Touch Log, C014 Contact Registry.
- **Reasoning**: C008 Refresh Engine, C009 Assessment Engine, C011 Synthesis Engine.

Read direction: Reasoning reads State, writes State, hands results
to the Renderer. Reasoning never writes back to sources.

## Things Explicitly Held for ADR

These were named in the architecture but deferred so the architecture
layer stays high-level:

- SQLite driver choice (`modernc.org/sqlite` pure-Go vs.
  `mattn/go-sqlite3` CGO).
- TOML library, OS keyring library, Jira API client library.
- Schema details for any State-band component (Item Store identity
  model, Override Store parameterization, Touch Log labels, etc.).
- Multi-source picking rule for the per-item "underlying-data last
  changed" timestamp under PRD002 (max-of, per-source, composite).
- Whether `birddog checkpoint` ever grows an interactive TUI.
- Exact CLI verb syntax, flag layout, and idempotence rules
  (`source add` when the source already exists: error vs. no-op).

## Pressure Points Flagged for the Judge Session

- **`J001-O004-R002` ownership.** The architecture maps R002 to "no
  new component beyond C015 + signal-producing components." A
  judge could push that R002 deserves an explicit owner — e.g., a
  "List Composer" component that picks which signals appear in
  which row at render time. Worth pressure-testing before treating
  the call as settled.
- **C007 (Item Store) breadth.** It currently absorbs item identity,
  active set, manual items, migration links, trajectories,
  dependencies, drop-offs, and rejected coverage candidates. A
  judge might argue that's too much for one component and push for
  splitting (Identity, Active Set, Dependency Graph, Coverage
  History). I held it as one to keep the architecture layer
  high-level; the components pass should reconsider.
- **Coverage Gap Discovery (`J001-O002-R003`) attribution.** Mapped
  to C008 + C007. The R-doc emphasizes a memory shape for rejected
  candidates that doesn't re-surface absent material change — that
  memory is in C007 today but might want explicit naming.

## Cross-Outcome Threads the Architecture Already Honors

For the judge to verify these are honored, not just claimed:

- **PRD001 (coverage over precision)** — every State-band store has
  explicit room for "we don't know" / "evidence disagrees" /
  "cannot assess" states. C009's responsibility paragraph names
  this directly.
- **PRD002 (two-timestamp freshness)** — set as a State-band
  invariant in the principles. The set-level refresh marker lives
  on C007's per-hunt active set; per-item timestamps live on each
  item record.
- **PRD003 (shared source registration)** — `J001-O001-R007` and
  `J001-O002-R007` map to the same component set (C004, C005,
  C003, C002). One implementation, two requirement-layer entries.
- **PRD004 (override surfacing)** — one C010 Override Store
  parameterized by which produced judgment it covers (status,
  intervention, escalation timing). Not three separate override
  mechanisms.

## Suggested Next Steps

1. **Judge session** on `docs/engineering/README.md` against the
   `Architecture` rubric in `intent/SKILL.md` (and `Records` if a
   PDR or ADR comes out of the discussion). Use the pressure
   points above as starting probes.
2. **Components pass** — author each `docs/engineering/components/
   C<NNN>-<name>.md`. Likely candidates for the simple template:
   C002, C003, C006, C012, C013, C014, C015. Likely candidates for
   the nested template (because they'll spawn ADRs early): C005
   Source Adapter, C007 Item Store, C008 Refresh Engine, C009
   Assessment Engine, C011 Synthesis Engine.
3. **First ADRs** likely to land alongside the components pass:
   - SQLite driver choice (CGO vs pure-Go) — probably under C007.
   - Multi-source underlying-data-change picking rule — under C009.
   - TUI deferral or adoption for `birddog checkpoint` — under C015.
   - Source Adapter contract shape (capabilities the Jira impl
     exposes; what a future source type must implement) — under
     C005.

## Open Questions Carried Forward

Not blocking the architecture, but live:

- Whether `bird-dogger` (product term) and `birddog` (binary name)
  staying distinct is desirable, or whether engineering should
  normalize on one. Currently both appear: product docs say
  "bird-dogger"; engineering says "birddog" for the binary and
  "bird-dogger" for the user. Probably fine as-is.

# ADR005 - Local-JSON Source Type

`local-json` is a first-class source type backed by a local file. It exercises the same `C005` Source Adapter contract as Jira, requires no token, and reports availability from the file's read state. It serves testing, demo, and fully-local workflows on equal footing with remote source types.

## Context

[Engineering README](../README.md) commits to a source-type-agnostic `C005` Source Adapter contract: the contract must stay source-type-agnostic so future source types (other trackers, doc systems, message platforms) plug in without changes to the Reasoning band. Jira-flavored adapters are the only concrete realizations declared so far — `jira-cloud` (Atlassian-hosted) and `jira-dc` (self-hosted Data Center) are two distinct source types but a single adapter family at the wire level (same query semantics, different auth and endpoint conventions). They do not exercise the contract's source-type-agnostic claim because they share too much.

The contract claim is testable only when a second, genuinely different adapter exists. Until then, "source-type-agnostic" is an aspiration that ships untested: implementation details of the Jira adapter family (auth shape, pagination, error model, availability semantics) tend to bleed into the contract without a sibling adapter to push back.

Three concrete use cases pull in the same direction:

- **Testing.** Reasoning-band components (`C008`, `C009`, `C011`) need deterministic input to test against. A file-backed source produces reproducible item sets without network mocks, fixture servers, or Jira test instances.
- **Onboarding and demo.** A bird-dogger evaluating the product, or a developer trying the chase loop end-to-end, should not need a Jira instance and an API token before they can run `bdog hunt bugel`.
- **Fully-local workflows.** Some bird-doggers track items the team maintains in a flat file, a shared spreadsheet exported as JSON, or a manually curated list. These items deserve the same coverage, assessment, and triage surface as Jira-backed items, not a separate manual-only side door.

The third case is where `local-json` differs from a hidden test fixture: it is a legitimate production source for some workflows. A test-only build tag would foreclose that use without saving the adapter design any cost.

Without a recorded decision, `local-json` either ships as a documented-but-undeclared backdoor (special-cased in the secret model, absent from the README's source-type list, hard to reason about) or gets refused on grounds that "real" sources should be the only ones, and the contract-agnosticism claim stays untested.

## Options

- **A. Add `local-json` as a first-class source type.** Same declaration shape as Jira (`bdog source add <name> --type local-json --path <file>`), same lifecycle, same source-availability reporting, no token. `C005` exposes the same read contract; the local-json implementation reads the file and translates to the adapter's item shape. Cost: one more adapter to maintain; one more declared type to document; file-schema decisions become public API.
- **B. Don't ship a local adapter.** Only support real-network source types. Cost: contract-agnosticism stays untested until a second remote adapter lands; testing pays for a fixture-server or network-mock layer; onboarding needs a Jira instance.
- **C. Test-only embedded fixture.** Ship a file-backed adapter behind a build tag (`-tags test`) or an internal flag, not exposed via the source-type enum. Cost: production users with a hand-curated file workflow have no path; the adapter exists but is hidden from documentation and CLI completion; the contract-agnosticism claim is tested only in test builds, leaving production builds running an untested abstraction.

## Decision

Option A — `local-json` as a first-class source type.

A second concrete adapter is the only honest test of the source-type-agnostic contract claim, and `local-json` is the cheapest second adapter available — no network, no auth, no rate-limit model, no pagination, no source-side workflow semantics. If the contract holds for both Jira and a file, it has a real chance of holding for the next tracker, doc system, or message platform.

The bird-dogger workflows it enables (fully-local item lists, demo/onboarding without an external instance, scriptable testing) are real product value, not test scaffolding. Hiding the adapter behind a build tag would make the production contract demonstrably less tested than the test contract, which inverts the usual ratio.

The auth model stays uniform across types under Option A: a source declares whether it needs a secret as part of its type. `local-json` declares none; Jira declares one. `C003` Credential Store is asked only for the secrets that exist; there is no special case at the keyring layer.

## Consequences

- `C005` Source Adapter contract is exercised by two distinct adapters from day one. The contract carries no Jira-specific assumptions about auth (`local-json` has none), pagination (`local-json` reads the whole file), rate limiting (`local-json` has none), or write capability (neither writes, per [ADR003](ADR003-read-only-against-sources.md)).
- `bdog source add --type local-json --path <file>` declares a local-json source. `--path` replaces `--url`; no token is prompted; no `BDOG_SOURCE_<NAME>_TOKEN` env-var path applies. The CLI design doc owns the verb shape; this ADR commits to the type's existence and the no-token property.
- `C003` Credential Store: `local-json` source declarations record no secret reference. `C003` is not asked for a secret when the adapter doesn't carry one. The keyring layer stays type-agnostic; the per-type "has a secret?" answer lives with the type declaration.
- Source availability for `local-json`: file exists and is readable → available; file missing, permission denied, or unparseable → unavailable. The unavailability surfaces through the same `J001-O001-R006` Source Availability path Jira uses; per [PRD001](../../product/pdrs/PRD001-coverage-over-precision.md), unreachable surfaces explicitly rather than as silence.
- File schema is the adapter's concern. The contract requires the adapter to emit items in `C005`'s item shape; how the JSON file expresses each item (field names, date formats, status vocabulary) is an adapter-level decision documented alongside the adapter, not at the `C005` contract layer. A minimal acceptable schema and its versioning policy are component-level ADR territory under `C005`.
- Item identity for local-json items uses the same scheme as every other source: `bdogitem-N` native identifier with the file-provided id available as the optional source id. Collision behavior across source types is unified.
- `bdog source test <name>` for a local-json source validates that the file exists, parses, and produces well-shaped items. The same verb covers both adapter types; the implementation differs.
- A future YAML-backed or CSV-backed adapter follows the same pattern: declare the type, take `--path`, no token, file-state availability. ADR territory if and when those land; this ADR sets the precedent.
- Composes with [ADR003](ADR003-read-only-against-sources.md): `local-json` is read-only against the file. The product does not write back to the file; the bird-dogger maintains the file out-of-tool.
- Composes with [ADR002](ADR002-on-demand-cadence-no-daemon.md): local-json refreshes happen inside a CLI invocation like every other refresh. There is no file-watcher mode.

# ADR007 - No-Cascade Removal

Removing a declarative entity or parent-of-references requires the
bird-dogger to tear down references from consumers first, then remove
the dependency. The CLI rejects removal with a list of blocking
references and the verbs that would clear them. Cascading deletes are
rejected because they would silently destroy declaration chains and
per-item history.

## Context

The bird-dogger manages a graph of declarative entities — sources,
selectors, contacts, chains, and hunts — that reference one another.
Items accumulate history tied to hunts and selectors. The TOML config
file captures these declarations and cross-references; SQLite stores
per-item state keyed on those declarations.

When a bird-dogger wants to remove an entity, two behaviors are
possible:

1. **Cascade:** The CLI removes the target entity and silently deletes
   every dependent entity and its history.
2. **Fail fast:** The CLI refuses removal when consumers still reference
   the entity, lists the blocking references, and tells the bird-dogger
   how to clear them.

There is a deliberate asymmetry in removal behavior. Source-derived
items lose scope links when a selector or hunt reference is removed
and **`item_selectors`** reconciles on the next refresh — the item row
remains in **Item Store (C007)**. Manual items have no source lifecycle;
every **`item_hunts`** row for the hunt must be cleared before the hunt
can be removed (multi-hunt manual items may retain links to other hunts).

Without a recorded decision, implementations might default to cascade
for convenience and produce silent data loss. The bird-dogger has no
recovery path once a declaration chain is destroyed in bulk.

## Options

- **A. Silent cascade.** Removing a source cascades to its selectors,
  then to hunt memberships, then to item coverage. One command tears
  down the entire graph. Cost: the bird-dogger loses per-item history,
  chain assignments, and override records without confirmation; there
  is no audit trail of what was removed or why; partial failures leave
  the graph in an inconsistent state.

- **B. Fail fast (no cascade).** Removal is rejected when any consumer
  still references the entity. The error names every blocking reference
  and provides the exact verbs to clear them. The bird-dogger controls
  the teardown sequence step by step. Cost: multi-step teardown for
  large graphs; the bird-dogger must run several commands to remove a
  source with many selectors and hunt memberships.

- **C. Interactive prompt.** On removal, the CLI lists all downstream
  entities and asks for confirmation before cascading. Cost: adds
  interaction to a non-interactive-friendly CLI; confirmation dialogs
  are skipped in scripts; per-entity history loss still happens on
  confirm; the bird-dogger cannot selectively preserve part of the
  cascade.

## Decision

Option B — fail fast, no cascade.

Cascading deletes (A) and confirmation prompts (C) both produce the
same outcome: one verb can silently or semi-silently destroy chains of
declarations and per-item history that the bird-dogger may want to
preserve or re-route. The fail-fast posture keeps the bird-dogger in
control at every step of the teardown. Each blocking reference surfaces
immediately with the exact verb to clear it, so the sequence is
visible, auditable, and reversible at each step.

The asymmetry between source-derived and manual items is deliberate.
Source-derived items are managed by the source; removing a selector from
a hunt or deleting the selector drops scope via **`item_selectors`**
reconciliation, but the item row remains in **Item Store (C007)**. Manual
items carry no source lifecycle — clear every **`item_hunts`** row for the
target hunt (via `item hunt remove` per item, or `item remove` to delete
the item entirely) before the hunt can be removed. A manual item in
multiple hunts keeps its other **`item_hunts`** rows untouched.

## Consequences

- `remove` on any declarative entity checks for outstanding references
  before acting. If any consumer still references the entity, the
  command fails with a list of blocking references and the verbs that
  would clear them.

- The required pre-step for each entity type:

  | Removing | Required pre-step |
  |---|---|
  | `source` | Remove or re-source every selector that references it. |
  | `selector` | Remove the selector from every hunt that references it (via `hunt selector remove`). |
  | `contact` | Remove from every chain that references it (via `chain member remove`) and clear any item owner reference (via `item set --owner none`). |
  | `chain` | Clear the chain reference from every item that references it (via `item set --chain none`). |
  | `hunt` | Clear all **`item_hunts`** rows for this hunt — `item hunt remove` for each manual item, or `item remove` to delete the item. Source-derived scope for the hunt clears when hunt selector references are removed; no manual unlink for source-backed items. See [ADR012](ADR012-scope-via-selectors.md). |
  | `item` (manual) | None; manual items have no downstream references. (Source-derived items cannot be removed individually — they are managed via the owning selector.) |

- `remove` is not idempotent for declarative entities: removing a
  non-existent or already-removed entity errors with `not found`.

- The teardown of a source with many selectors and hunt memberships
  requires several commands. This cost is accepted because the
  alternative — bulk silent deletion — is unrecoverable.

- See [CLI (C001)](../components/C001-cli.md#behavior) "Removal semantics" and the
  [`Tearing down a source (no-cascade)`](../components/C001-cli.md#tearing-down-a-source-no-cascade) example for the full
  per-noun verb table and a worked teardown walkthrough.

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
items disappear automatically when their owning hunt or selector is
removed — the source manages their lifecycle. Manual items have no
source to clean them up; they require an explicit reassignment or
removal step before the owning hunt can be removed.

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
Source-derived items are managed by the source; removing the owning
hunt or selector removes the context that makes them live, but the
underlying item records remain until the source is removed or the
selector stops matching them. Manual items carry no source lifecycle —
they must be explicitly re-assigned (via `item set --hunt <other>`) or
removed (via `item remove`) before the owning hunt can be removed.

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
  | `hunt` | Reassign or remove every manual item whose `--hunt` is this hunt (via `item set --hunt <other>` or `item remove`). Source-derived items disappear from coverage automatically. |
  | `item` (manual) | None; manual items have no downstream references. (Source-derived items cannot be removed individually — they are managed via the owning selector.) |

- `remove` is not idempotent for declarative entities: removing a
  non-existent or already-removed entity errors with `not found`.

- The teardown of a source with many selectors and hunt memberships
  requires several commands. This cost is accepted because the
  alternative — bulk silent deletion — is unrecoverable.

- See [cli.md](../design-docs/cli.md) "Removal semantics" and the
  `### Tearing down a source (no-cascade)` example for the full
  per-noun verb table and a worked teardown walkthrough.

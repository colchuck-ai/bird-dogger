# Component documentation template

Every component doc under `docs/engineering/components/` follows the same section shape. Use [CLI (C001)](C001-cli.md) and [Renderer (C015)](C015-renderer.md) as exemplars when authoring or reviewing a component page.

## Required sections (in order)

Write these as `##` headings. Do not skip a section — use a one-line placeholder when a component has nothing substantive yet (e.g. "None identified for v1.").

| # | Section | Purpose |
|---|---------|---------|
| 1 | **Data model** | Persistent shapes, ids, storage keys, invariants the component owns or exposes. |
| 2 | **Interfaces** | External surfaces: CLI verbs, function contracts, config tables, renderer columns, etc. Component-specific subsections are fine (see C015 *Renderer surfaces*). |
| 3 | **Behavior** | Runtime behavior, orchestration, validation rules, and dispatch paths. |
| 4 | **Edge cases** | Failure modes, ambiguity, idempotency, and boundary conditions worth calling out explicitly. |
| 5 | **Relationships** | Upstream/downstream components and what crosses each boundary (read, write, invoke). |
| 6 | **Success criteria** | Testable "done" statements for this component — observable outcomes, not implementation steps. |
| 7 | **Notes** | Design decisions, rationale, and non-obvious trade-offs. Use `### Design decisions` (and other subsections) for longer notes, as in C001. |
| 8 | **Open decisions** | **Required.** Items still in ADR territory or otherwise unresolved. List each open question with enough context that a future ADR or doc pass can pick it up. Use "None — all decisions recorded in linked ADRs." only when every remaining choice is already captured in `docs/engineering/adrs/`. |

## Section flow

```
Data model
  → Interfaces
  → Behavior
  → Edge cases
  → Relationships
  → Success criteria
  → Notes (design decisions)
  → Open decisions
```

## What belongs here vs elsewhere

- **Template only.** This page defines section structure; it does not invent component behavior.
- **ADRs** record committed cross-cutting decisions; link them from *Notes* or *Open decisions* rather than duplicating full ADR text.
- **Engineering README** holds the architecture overview and per-component summaries; deep detail lives in the component file.

## Exemplars

| Doc | Why it is an exemplar |
|-----|------------------------|
| [C001-cli.md](C001-cli.md) | Full stack: data model, large *Interfaces* verb catalog, *Behavior*, *Edge cases*, *Relationships*, *Success criteria*, *Notes* with *Design decisions* and worked examples. |
| [C015-renderer.md](C015-renderer.md) | Component-specific *Interfaces* subsections (*Readability obligations*, *Renderer surfaces*) while keeping the same top-level flow through *Open decisions*. |

When adding a new `C0xx-*.md` file, copy the heading skeleton from an exemplar, fill each section, and ensure **Open decisions** is present before marking the doc complete.

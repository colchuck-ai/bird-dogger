# ADR011 - Two Clearing Shapes (`none` vs `--clear-<field>`)

Fields on `set` verbs are cleared by one of two shapes, determined by
field type: typed reference fields clear with the sentinel `none`
(`--chain none`, `--owner none`); free-text fields clear with an
explicit flag (`--clear-email`, `--clear-pace`). The rule is uniform
across every `set` verb that has clearable fields.

## Context

Several `set` verbs allow a field to be cleared — returning it to an
unset state — without removing the parent record. Two families of
clearable fields exist:

**Typed reference fields** accept an entity name as their value. The
bird-dogger-chosen entity namespace reserves `none` as a keyword: no
source, chain, contact, hunt, or selector can be named `none`. Fields
in this family include `--chain` and `--owner` on `item set`.

**Free-text fields** accept arbitrary strings. Channel handles
(`--email`, `--slack`, `--jira`), descriptions (`--about`), and pace
text (`--pace`) could legitimately hold the string `"none"` as a real
value. Treating `none` as a sentinel here would make it impossible to
set a free-text field to the string `none`.

Without a recorded rule, each verb author makes an ad-hoc choice. One
author uses `--chain none`; another adds `--clear-email`; a third adds
a `--no-pace` boolean. The bird-dogger has to memorize per-field
clearing behavior across the entire verb surface rather than applying
one rule.

## Options

- **A. Sentinel `none` for all clearable fields.**  
  `bdog item set <item> --chain none` and `bdog contact set alice
  --email none` both clear their respective fields. Cost: a contact
  (or other entity) legitimately named `none` becomes impossible; a
  free-text field value equal to the string `"none"` is
  unrepresentable; the rule is simple to state but unsafe to generalize.

- **B. `--clear-<field>` flags for all clearable fields.**  
  Every clearable field has a paired boolean flag: `--clear-chain`,
  `--clear-owner`, `--clear-email`, `--clear-pace`. Cost: typed
  reference fields already have a natural sentinel (`none` cannot name
  any real entity); a parallel `--clear-*` flag is extra surface for
  no safety benefit; the combined flag set doubles the number of
  clearing-related flags on each verb.

- **C. Sentinel `none` for typed references; `--clear-<field>` for
  free-text.**  
  The shape follows the field's value space: typed reference fields
  use the reserved sentinel; free-text fields use an explicit clearing
  flag. The bird-dogger learns one rule tied to observable field
  semantics rather than per-field convention.

## Decision

Option C — clearing shape chosen by field type.

The sentinel `none` is safe precisely where entity names are
constrained: the namespace reserves `none` as a keyword, so it cannot
collide with a real chain name, contact name, or selector name. Typed
reference fields (`--chain`, `--owner`) therefore accept `none` without
ambiguity — the token can only mean "clear this field."

Free-text fields carry no such constraint. A channel handle, a
description, or a pace string that happens to equal `"none"` is a valid
value that the bird-dogger might intentionally set. Reserving `none` as
a sentinel would silently corrupt any such value or require escaping
that the bird-dogger would not expect. An explicit `--clear-<field>`
flag makes the intent unambiguous and leaves the value space of the
field fully open.

The rule pairs the clearing shape to the observable property of the
field — whether its target is a constrained entity namespace or
unconstrained free text — rather than to the verb, the noun, or an
ad-hoc per-field decision. Any future clearable field falls into one of
the two categories; no case-by-case ruling is needed.

`--clear-*` flags and their value-bearing siblings are mutually
exclusive on the same call: `--email foo --clear-email` is an error.

## Consequences

- Every `set` verb with a typed reference field exposes `--<field>
  <name>|none`. Supplying `none` clears the field; supplying an entity
  name assigns it. No additional flag is needed.
- Every `set` verb with a free-text field exposes `--<field> <value>`
  for setting and `--clear-<field>` for clearing. The string `"none"`
  remains a legal value for all free-text fields.
- The current instances:

  | Verb | Field | Clearing shape |
  |---|---|---|
  | `item set` | `--chain` | `--chain none` |
  | `item set` | `--owner` | `--owner none` |
  | `item set` | `--pace` | `--clear-pace` |
  | `contact set` | `--email` | `--clear-email` |
  | `contact set` | `--slack` | `--clear-slack` |
  | `contact set` | `--jira` | `--clear-jira` |
  | `contact set` | `--about` | `--clear-about` |

- `--trajectory none` on `item set` is a special case: it clears both
  `--due` and `--pace` as part of the trajectory cross-validation
  rule. `--trajectory` is a closed-vocabulary enum, not an entity
  reference, but `none` is a legal enum member that carries the
  "clear" semantic. This is consistent with Option C: the value space
  of `--trajectory` is controlled (four fixed values), so `none` cannot
  collide with an arbitrary user string.
- Future clearable fields follow the same rule. The verb author asks:
  "is this field's value constrained to an entity namespace or a
  controlled vocabulary?" If yes, add `|none` to the flag; if no, add a
  `--clear-<field>` flag.
- Composes with
  [ADR010](ADR010-tokens-never-via-cli-flag.md): token fields are
  not clearable via any flag — neither `--token none` nor
  `--clear-token` exists. Credential removal happens by removing the
  source declaration (`source remove`). ADR010 forecloses the flag
  shape entirely; this ADR does not create an exception for credential
  fields.
- See [cli.md](../design-docs/cli.md) §"Clearing a field" (near the
  top of the Patterns section) and the `contact set` and `item set`
  verb definitions for the canonical examples.

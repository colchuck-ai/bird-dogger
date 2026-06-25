# Touch Log (C013)

Append-only log of bird-dogger contact events on items — routine check-ins and escalation attempts — with a shared record shape, response state carried on each touch, and per-item response windows derived at read time from touch timestamps.

## Data model

SQLite-backed State-band store. One table owns all touch records; escalation attempts and routine touches share the same schema and differ only by the `type` label.

### `touches(id, item_id, type, channel, target, message, at, response_text, responded_at)`

| Column | Role |
|--------|------|
| **`id`** | Native touch id `bdogtouch-<n>` per [ADR006](../adrs/ADR006-native-id-scheme.md); monotonically assigned on `add`. |
| **`item_id`** | References Item Store (C007) `items.id` (`bdogitem-<n>`). |
| **`type`** | Touch kind — `escalation` or `checkin` at the CLI surface ([C001](C001-cli.md#touch)). Distinguishes escalation attempts from routine touches on the same row shape (J001-O005-R005). |
| **`channel`** | Free-text channel label (e.g. `slack`, `email`, `meeting`). Not normalized to an enum in v1. |
| **`target`** | Optional Contact Registry (C014) contact name — typically a chain member for escalations or the owner for a routine touch. Null when the bird-dogger omits `--target` and captures the person in `--message` instead. |
| **`message`** | Optional free-text content of the contact event (`--message`). |
| **`at`** | When the bird-dogger made the contact (`--at`, default now). Anchors response-window derivation. |
| **`response_text`** | Response body once recorded via `record-response`; null until then. |
| **`responded_at`** | When the response was received (`--responded-at` on `record-response`); null until then. |

### Invariants

- **Append-only writes.** New touches are inserted; existing rows are updated only through `set` (field correction) or `record-response` (response attachment). No in-place rewrite of history beyond those explicit verbs.
- **Hard-delete on `remove`.** `touch remove` permanently drops the row per [ADR008](../adrs/ADR008-history-posture-per-record-type.md) — touches are corrective records, not audit-trail policy.
- **Response window not stored.** Per-item elapsed time since the most recent touch (`now − max(at)` over touches for that item) is computed by readers (Synthesis Engine (C011), Renderer (C015)); Touch Log (C013) holds the anchor timestamps only (J001-O005-R003).
- **Out-of-tool send.** Touch Log (C013) records what the bird-dogger did; the product does not send email, Slack, or source comments ([ADR003](../adrs/ADR003-read-only-against-sources.md)).
- **Item foreign key.** Every `item_id` must resolve to an existing Item Store (C007) row at write time.

### Derived read: response window

For item I at read time T:

```
response_window(I, T) =
  if no touches for I: absent ("no touch recorded")
  else if most_recent_touch(I) has responded_at set: closed (response visible alongside window)
  else: T − at(most_recent_touch(I))   # elapsed since last touch
```

When multiple touches exist on the same item, the most recent by `at` anchors R003's list-level window; per-target history remains in the full touch list (R005).

## Interfaces

Touch Log (C013) has no standalone CLI surface. [CLI (C001)](C001-cli.md#touch) dispatches touch verbs; other components read through internal contracts.

### Write path (via CLI → C013)

| CLI verb | Effect on C013 |
|----------|------------------|
| `bdog touch add <item> --type … --channel … [--target …] [--message …] [--at …]` | Insert row; print assigned `bdogtouch-<n>`. |
| `bdog touch set <id> --channel \| --target \| --at \| --type \| --message` | Update listed fields on existing row. |
| `bdog touch record-response <id> --response … --responded-at …` | Set `response_text` and `responded_at`; side-effecting verb, not a `set` flag. |
| `bdog touch remove <id>` | Hard-delete row; errors if not found. |

### Read path

| Consumer | Reads |
|----------|---------|
| CLI (C001) | `info`, `list` with filters `--item`, `--since`, `--channel`, `--type`, `--open` (touches without a recorded response). |
| Synthesis Engine (C011) | Prior attempts and derived response window for escalation-timing recommendation (J001-O005-R004). |
| Renderer (C015) | Touch history and derived response window on item detail; recent touches (default last 5, `--show-history` for all). |

## Behavior

### Recording a touch

On `touch add`, CLI (C001) validates the item exists in Item Store (C007), resolves optional `--target` against Contact Registry (C014), assigns the next `bdogtouch-<n>`, and inserts the row. The bird-dogger sends the actual message out-of-tool; the row records intent and metadata only ([ADR003](../adrs/ADR003-read-only-against-sources.md)).

`type=escalation` and `type=checkin` use identical columns. Escalation attempts are not a separate store — the label satisfies R005's distinction while keeping R003's anchor shape shared.

### Recording a response (`record-response`)

`record-response` is its own verb (not folded into `set`) because it advances response state on an existing touch ([C001](C001-cli.md#behavior) "Side-effecting actions on own verbs").

| Re-invocation | Behavior |
|---------------|----------|
| Identical `--response` and `--responded-at` | Idempotent no-op. |
| Different response or timestamp | Overwrites prior response fields. |

After a response is recorded, readers treat the item's response window as closed or reset and surface the response alongside the item (J001-O005-R003 acceptance).

### Listing and filtering

- **`--open`:** touches where `responded_at` is null — outstanding contact events.
- **`--since`:** touches with `at` on or after the filter timestamp.
- Default sort for item-scoped lists: newest `at` first (consistent with Renderer (C015) recent-touches surface).

### Removal

`touch remove` hard-deletes the row. Not idempotent: missing id errors with not found ([ADR008](../adrs/ADR008-history-posture-per-record-type.md)). Removing the most recent touch changes derived response-window results on the next read — by design, for mistake correction.

## Edge cases

- **No touches on item:** Response window surfaces as absent / "no touch recorded", not zero elapsed (PRD001 / J001-O005-R003).
- **Unregistered target:** Row is still stored; `target` is null and the person may appear only in `message` ([C001](C001-cli.md#edge-cases) "Unregistered touch target").
- **Response on a different channel than the attempt:** Recorded as-is on the same touch row; channel mismatch is preserved, not normalized (J001-O005-R005).
- **Failed delivery (bounced email, undelivered DM):** Recorded with explicit failure state in `response_text` / response semantics per PRD001 — not omitted or normalized to success (J001-O005-R005).
- **Multiple recent touches:** R003 uses elapsed since the most recent touch only; full per-target history remains in the touch list for R005/R004.
- **Item falls out of scope:** When an item loses all scope links, Item Store (C007) and dependent touch rows remain ([ADR007](../adrs/ADR007-no-cascade-removal.md)); touches are still readable if the item id is known.
- **`touch set` after `record-response`:** Field edits are allowed for correction; re-running `record-response` with different input overwrites response fields per idempotence table.

## Relationships

- **CLI (C001)**: sole writer — dispatches `touch add`, `set`, `record-response`, `remove`, `info`, `list`.
- **Item Store (C007)**: `item_id` foreign key; every touch belongs to one item.
- **Contact Registry (C014)**: optional `target` resolves to a registered contact name (chain member or owner).
- **Synthesis Engine (C011)**: reads prior attempts and derived response window for escalation-timing recommendation; does not write C013.
- **Renderer (C015)**: reads touch history and derived response window for list and item detail surfaces.
- **Note Store (C012)**: sibling State-band per-item history; notes capture triage commentary with context snapshots — distinct from contact events in C013.

## Success criteria

- Doc states append-only insert semantics with explicit update paths (`set`, `record-response`) and hard-delete on `remove`.
- Shared row shape for `escalation` and `checkin` documented; `type` is the sole discriminator.
- Response window documented as derived at read time from touch `at` timestamps, not stored.
- `record-response` idempotence matches [C001](C001-cli.md#behavior) behavior table: idempotent on identical input, overwrite on different input.
- Out-of-tool send posture linked to [ADR003](../adrs/ADR003-read-only-against-sources.md); C013 records attempts, not sends.
- R003 and R005 requirements traceable: last-touch anchor, response visibility, escalation history at item level.

## Notes

### Design decisions

**One store, two roles (R003 + R005).** Routine check-ins and escalation attempts share `touches` because R005 specifies one record shape with a label distinction. R003 reads the most recent touch regardless of type; R004 reads attempt history filtered by context. A split store would duplicate timestamps and complicate "most recent touch" queries.

**Response window as derivation, not column.** Storing elapsed time would stale immediately and fork logic when clocks or touch rows change. Readers compute `now − at` (or "closed" when `responded_at` is set) so list and detail views stay consistent with the log ([engineering README](../README.md) C013 summary).

**Hard-delete posture.** Unlike Override Store (C010), touches carry no PRD commitment to preserved history after removal. Fat-fingered targets or duplicate entries should disappear entirely ([ADR008](../adrs/ADR008-history-posture-per-record-type.md)).

**Escalation as touch kind, not separate component.** `--type escalation` on `touch add` is the CLI surface for recording an escalation attempt; there is no Escalation Store. Synthesis and Renderer consume the same rows as routine touches.

## Open decisions

- **Response state vocabulary.** J001-O005-R005 names response states (received, none yet, declined, redirected) at the product layer; v1 encodes response in free-text `response_text` plus `responded_at` rather than a closed enum. A future ADR may normalize response states if synthesis or rendering needs structured filtering.
- **Channel taxonomy.** `--channel` is free text at the CLI; normalization (canonical channel names, delivery-failure markers) remains open unless a product PDR closes it.
- **Retention horizon.** R005 does not prescribe how long touch history is kept; v1 assumes indefinite SQLite retention until explicit `remove`. Archival or pruning policy is unset.

# ADR002 - On-Demand Cadence, No Daemon

Refresh, re-assessment, and synthesis run inside a `bdog hunt bugel`
(or scoped equivalent) invocation. Nothing runs in the background;
there is no daemon, no scheduler inside the product, no notification
subsystem.

## Context

The chase loop runs from one checkpoint to the next (product README,
J001 narrative). A checkpoint is the bird-dogger sitting down —
explicitly — to review the active set and decide what needs
intervention. Several requirements describe work that has to happen
"before each checkpoint" or "on demand":

- `J001-O001-R001` Proactive Slip Surfacing — items appear to be
  slipping without per-item inspection.
- `J001-O002-R006` Active Set Freshness — the active set is refreshed
  before each checkpoint and the last-refresh time is shown.
- `J001-O003-R005` Re-Assessment On Change — items whose underlying
  data has changed get re-assessed.
- `J001-O004-R001` Urgency Ranking, `R003` Intervention
  Recommendation, and `J001-O005-R004` Escalation Timing
  Recommendation — synthesized over current State on the
  bird-dogger's request.

None of those R-docs use "push" or "notify"; each can be honored by
computing the right answer when the bird-dogger asks. But
`J001-O001-R001`'s "without inspecting each item" can fairly be read
as "the product tells me proactively," which a true push-notification
architecture would satisfy more directly than a checkpoint pull.

The constraints push the other way: local-only state on the
bird-dogger's host (no server, no shared state), single user per
host, single static binary distributable via `go install`, no
installer, no separate runtime.

## Options

- **A. On-demand cadence, no daemon.** Every refresh, re-assessment,
  and synthesis pass runs inside a CLI invocation — `bdog hunt
  bugel` for the full pass, `bdog hunt bugel --no-refresh` to
  re-synthesize against current State without pulling sources, and
  `bdog item info <item>` for per-item inspection. No process runs
  between invocations. The bird-dogger drives the schedule via their
  own habits, calendar, cron, launchd, systemd timers, or whatever
  they already use. Cost: `J001-O001-R001`'s "without inspecting each
  item" obligation is satisfied at the next bugel, not before; a slip
  beginning five minutes after a bugel waits until the next one to
  surface.
- **B. Background daemon.** A long-running `bdog` process polls
  sources on a schedule, runs assessment and synthesis as data
  arrives, and notifies the bird-dogger when items cross thresholds
  (OS notification, terminal bell, email, etc.). Cost: a network
  listener (or at least an outbound notification surface), a service
  to install and supervise per OS (launchd / systemd / Windows
  Service), state-sync questions between the daemon and CLI
  invocations (who owns the SQLite write lock?), a separate auth
  model for the notification channel, error-recovery behavior when
  the daemon crashes silently between checkpoints. The
  single-static-binary distribution constraint and the local-only /
  no-server constraint both bend or break.
- **C. Hybrid — optional daemon, on-demand default.** Ship as
  on-demand by default; offer an opt-in daemon mode for bird-doggers
  who want push notifications. Cost: two architectures to maintain,
  two test matrices, two failure modes; the daemon's state-sync
  questions still need answers because CLI invocations must keep
  working alongside it.

## Decision

Option A — on-demand cadence, no daemon.

The product's whole posture is local-only, CLI-first, one-shot
subcommands (engineering README principles). A daemon would
re-introduce a server-shaped subsystem (background process,
notification channel, supervision, state-sync) that every other
architectural decision explicitly excludes. The bird-dogger's natural
cadence is already discrete (the checkpoint is a moment, not a
stream), and operating systems already provide perfectly good
schedulers (cron, launchd, systemd timers) that can call `bdog hunt
bugel` on whatever rhythm the bird-dogger wants — the product does
not need to ship its own.

The cost is real and worth naming: `J001-O001-R001`'s "surface
slipping items without inspection" is honored at the next bugel, not
the instant the slip begins. For the chase-loop framing — sit down
for a bugel, review, intervene, move on — that lag is acceptable. A
bird-dogger who wants tighter cadence schedules more bugels; the
product does not try to be a pager.

The hybrid option is rejected because it carries the daemon's full
cost (long-running process, notification channel, state-sync model)
without the offsetting simplicity of either pure option.

## Consequences

- No network listener, no notification subsystem, no service-installer
  flow. Distribution stays `go install` plus a future Homebrew tap.
- SQLite's single-writer model fits cleanly: every write happens
  inside a CLI invocation; concurrent invocations are rare and
  handled by SQLite's normal lock semantics. No daemon write-lock
  coordination needed.
- The bird-dogger owns the checkpoint cadence externally.
  Documentation should call this out: `cron` / `launchd` / `systemd`
  examples in the README rather than hiding scheduling inside the
  binary.
- `bdog hunt bugel` is the full chase pass — refresh → re-assess
  changed items → synthesize → render — and the only refresh path
  the bird-dogger invokes. Partial work is covered by `bdog hunt
  bugel --no-refresh` (skip the pull, re-synthesize current State)
  and by `bdog item info <item>` for per-item inspection; there is
  no standalone `bdog hunt refresh` verb. Exact verb syntax, flag
  layout, idempotence rules, and renames live in the
  [CLI design doc](../design-docs/cli.md).
- The two-timestamp freshness shape (PRD002) carries more weight
  under this decision than it would under a daemon: there is no
  notification telling the bird-dogger anything has changed since the
  last checkpoint, so the `last-assessed` and
  `last-underlying-data-change` markers are the only signals for
  "how stale is what I am looking at." `C015` and the principle
  wording in the engineering README already commit to surfacing them
  prominently.
- If a future requirement legitimately needs push behavior (e.g.,
  paging the bird-dogger when a critical item crosses a threshold
  between checkpoints), this ADR must be superseded with a new ADR
  that accepts the daemon-shaped costs explicitly. The current
  product framing does not include that requirement.
- Composes with
  [ADR003](ADR003-read-only-against-sources.md): no daemon means no
  background polling, which means source rate-limit pressure stays
  bounded to one burst per explicit checkpoint pull.

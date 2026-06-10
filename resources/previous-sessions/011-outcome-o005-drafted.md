# Session 011 — Drafted outcome O005 Escalation Quality

## Goal

Use the `intent` skill to draft the last outcome document under J001.
O005 Escalation Quality was the standing next pick from sessions 009
and 010 — chase-loop step four (Detect → Cover → Triage → Escalate),
the last undrafted outcome.

## Context loaded

- `docs/product/README.md` — current product root.
- `docs/product/outcomes/J001-O001-slip-detection-lag/README.md` —
  sibling; slip status is an input to escalation timing.
- `docs/product/outcomes/J001-O002-coverage/README.md` — sibling; owner
  identification boundary.
- `docs/product/outcomes/J001-O003-status-fidelity/README.md` —
  sibling; RSK002 boundary check against O003-RSK002.
- `docs/product/outcomes/J001-O004-triage-speed/README.md` — sibling
  immediately upstream; triage ends where escalation begins.
- `resources/previous-sessions/001` through `010` — locked decisions,
  deferred risks, standing fragilities, S002's standing flag that O005
  packs both targeting and timing.
- `resources/bird-dogging/bird-dogging-overview.md` — "Escalate when
  items stall or owners are unresponsive" as a core responsibility;
  cadence step 4 names escalation.
- `intent` skill: `SKILL.md` (Outcome / Risk / Requirement rubrics),
  `references/context-tracing.md` (Outcome trace), `references/
  workflows.md` (three-session model, create has no CR),
  `assets/templates/outcome.md`.

Skipped: Records (CRs, PDRs, ADRs) per workflows guidance — no record
history needed for a create.

## Workspace path mismatch (still standing from sessions 002–010)

Cursor opens `/Users/max.dunn/dev/personal/jomadu/bird-dog-skill`;
real repo is `/Users/max.dunn/dev/personal/colchuck-ai/
bird-dogging-skill` (resolved via symlink). Wrote against the real
path. Worth fixing the workspace mapping.

## Swap files

Author confirmed at start of session that vim swap files in O003 and
O004 folders (flagged sessions 004–010) have been handled. Standing
flag cleared.

## Outcome selection

Pre-decided by user prompt and by S009/S010's standing
recommendation. No alternative survey this session.

## Outcome framing decisions

S002's standing flag — O005 packs both *targeting* (right person) and
*timing* (right time) — was the one open framing call. Three options
presented to the user: (a) keep joint, (b) split into two outcomes,
(c) keep joint but subdivide risks into Targeting and Timing groups
in the doc.

User picked **keep joint**. Defense:

- Customer judges an escalation as good/bad as a whole. "Right person,
  wrong time" and "right time, wrong person" both feel like a failed
  escalation.
- "Confidence" sits on the escalation as a unit, not on dimensions.
- Other outcomes already span multiple failure-mode categories (O001
  covers both trajectory and silence).
- Splitting would require a product-root PDR, out of scope for a
  create cycle.

Drew the outcome line at "given an item has been triaged as needing
intervention now (O004), how good is the escalation?" O005 is the
*tail* outcome of the chase loop — it consumes triage's output and
asks whether the escalation it triggers reaches the right person at
the right moment.

Boundary against siblings:

- **vs O001 Slip Detection Lag.** O001 surfaces *that* slipping;
  slip status is an input to escalation timing (Late Escalation
  uses it).
- **vs O002 Coverage.** O002 keeps items on the list with an owner
  recorded. O005 presumes a target exists and asks whether it's the
  right one. "No owner at all" is O002 territory.
- **vs O003 Status Fidelity.** O003 makes the status trustworthy.
  RSK002 (Premature Escalation) is about the *owner's response
  window* after the bird-dogger's last touch; O003-RSK002 (Stale
  Status Assessment) is about the *item's status data* going stale.
  Different objects.
- **vs O004 Triage Speed.** O004 picks *which* item needs
  intervention now. O005 starts after that pick — how is the
  escalation executed?

## Scope decision

User answer to scope-check: "between 2 and 7 risks, bias minimal."

Resolved to 4 risks. Walked from a 6-risk candidate list (Wrong
Owner, Authority Chain Mismatch, Premature, Late, History Loss,
Channel Mismatch) down by:

- Folding Wrong Owner + Authority Chain Mismatch into **Misdirected
  Target** (RSK001) — both are "recorded target is wrong" with
  paired mitigations (R001 covers chain, R002 covers freshness).
  Precedent: O001-RSK001 (Coarse Check Cadence) is a single broad
  risk mitigated by R001 + R002.
- Cutting **Channel Mismatch** — folds into the "right person at
  right time" reading; the user did not specifically call it in.
  Standing flag for judge.
- Cutting **No Owner At All** — O002/O003 territory per the
  sibling-boundary call above.

Final 4 risks, each customer-centric, solution-free, and closing on
the outcome statement clause `...reducing confidence that escalations
land with the right person at the right time.`

## Risk framing decisions

- **RSK001 Misdirected Target.** Broad condition — recorded target is
  wrong, either because the owner is no longer the right person or
  because no escalation chain above the owner is recorded. Two
  failure conditions under one risk because the mitigations pair
  naturally (R001 = chain data, R002 = owner currency). Judge may
  try to split.
- **RSK002 Premature Escalation.** Timing relative to the *owner*.
  Escalation lands before the owner has had a fair window to respond.
  Distinct from O003-RSK002 — different objects (owner response
  window vs status data staleness).
- **RSK003 Late Escalation.** Timing relative to the *item*.
  Escalation lands after the intervention window has closed. Pairs
  with RSK002 as the two-axis timing failure under joint framing.
- **RSK004 Escalation History Loss.** Prior escalation attempts on
  an item not carried forward — bird-dogger may re-escalate a failed
  path or skip a step in the chain. Parallel shape to O004-RSK004
  (Triage Memory Loss): same item-state-loss-between-checkpoints
  pattern, distinct object (escalation history vs triage decision).

## Boundary calls worth flagging for the judge

- **RSK001 internal fold (stale owner + missing chain).** Could be
  split into two risks. Argued: both are "recorded target is wrong"
  with paired mitigations. Judge may try to split.
- **RSK002 (O005) vs O003-RSK002 (Stale Status Assessment).** Both
  about something going stale. Distinct *objects*: O003 is item
  status data; O005 is the owner's response window after the
  bird-dogger's last touch. Judge may probe.
- **RSK004 (O005) vs O004-RSK004 (Triage Memory Loss).** Same
  shape — state-loss-between-checkpoints — different objects.
  Mirrors the O004-RSK004 vs O002-RSK003 pattern resolved in S010.

## Requirement framing decisions

Five requirements, each tied to ≥1 risk. All behavioral.

- **R001 Escalation Target.** Surfaces the current owner and the
  escalation chain above them for each item. Names *what* (surface
  the chain), not *how* (org chart import, manual entry, sentiment
  analysis).
- **R002 Owner Currency.** Shows how recently each item's recorded
  owner was confirmed. Parallel to O001-R002 Assessment Freshness
  and O003-R002 Status Freshness.
- **R003 Response Window Visibility.** Shows how long the owner
  has had to respond since the bird-dogger's last touch on each
  item. Fact-surfacing for a timing input — bird-dogger uses it to
  judge prematurity.
- **R004 Escalation Timing Recommendation.** The most
  product-shaping requirement here. Mirrors O004-R003 Intervention
  Recommendation verbatim in shape — the product must commit to a
  notion of "escalate now" and apply it consistently across both
  timing axes, rather than leaving the synthesis entirely to the
  bird-dogger. Doubles as a partial mitigation of RSK002 — by
  shrinking the over-escalation surface — and primary mitigation
  of RSK003.
- **R005 Escalation Record.** Records each escalation attempt
  (target, channel, time, response) and surfaces that history at
  the item level. Parallel to O004-R004 Triage State
  Carry-Forward.

Risk-Requirement Map:

- RSK001 → R001, R002
- RSK002 → R003, R004
- RSK003 → R004
- RSK004 → R005

R004 carries two risks (RSK002 partial, RSK003 direct). Defensible:
RSK002 is "too soon" and RSK003 is "too late"; R004 supplies a
committed timing notion that addresses both directions. Mirrors
O004-R003's RSK001 + RSK003 doubling pattern; the S010 judge pass
upheld that pattern with the same defense.

All other mitigations are single. Pattern matches O001-RSK002 /
O002-RSK005 / O003-R005 / O004-RSK002/RSK004/RSK005.

## What got written

`docs/product/outcomes/J001-O005-escalation-quality/README.md` — full
outcome document. Outcome statement lifted verbatim from product
root for narrative coherence (same move as S3, S5, S7, S9). Four
risks, five requirements, risk-requirement map.

No Change Record — this is a create, per workflows.md (and the
S008 process slip lesson confirmed across S010).

## Coverage check

- Every risk has ≥1 requirement: ✓
- Every requirement has ≥1 risk: ✓
- All four risk impact clauses close on the outcome statement
  (`...reducing confidence that escalations land with the right
  person at the right time.`).
- **R004 carries 2 risks** (RSK002 partial + RSK003 direct).
  Defensible; mirrors O004-R003 pattern.
- **RSK003, RSK004 each have single-mitigation** (R004 and R005
  respectively). Standing fragility, parallel to O001-RSK002 /
  O002-RSK005 / O003-R005 / O004-RSK002/RSK004/RSK005. Not a
  defect.

## Deferred (so we don't lose it)

- **RSK001 internal fold pressure.** Could split into stale-owner
  and missing-chain risks. Argued: paired mitigations justify the
  fold. Judge may try to split.
- **R004 (Escalation Timing Recommendation) solution-talk
  pressure.** "Synthesize signals into a recommendation" mirrors
  O004-R003 verbatim. Same defense applies (the *what*, not the
  *how*); same engineering-layer mechanism choice (heuristic,
  scoring, local-LLM, rule set). Judge will probe; S010 upheld the
  pattern for O004-R003.
- **R004 doubling on RSK002 + RSK003.** Argued distinct mitigation
  paths (both timing directions); judge may try to pull R004 off
  RSK002 if R003 alone is enough.
- **RSK002 (O005) vs O003-RSK002 boundary.** Different objects
  (owner response window vs status data). Judge may probe.
- **RSK004 (O005) vs O004-RSK004 boundary.** Same pattern, different
  objects. Mirrors the S010 resolution of O004-RSK004 vs
  O002-RSK003.
- **No channel-mismatch risk.** Folded into "right person at right
  time" reading. Judge may want it pulled out as a separate risk or
  added under RSK001.
- **No no-owner-at-all risk.** Treated as O002/O003 territory.
  RSK001 presumes a target exists but is wrong. Judge may probe.
- **R001 / R005 introduce new product-held data.** Target/chain
  data (R001) and escalation history (R005) are data the product
  hasn't been asked to hold before. Sharpens engineering-layer
  concerns; not a product-layer defect.
- **RSK003 fragility under single-risk coverage.** New this session.
- **RSK004 fragility under single-risk coverage.** New this session.
- **Engineering concerns** from sessions 001 / 002 now sharpened by
  O005:
  - **Pluggable inputs** — R001 (Escalation Target) consumes
    org-chart / RACI / prior-escalation data. Adds to the
    pluggable-input pressure from O001-R007, O002-R002 / R005 /
    R007, O004-R002.
  - **Pluggable outputs** — R005 (Escalation Record) implies the
    product captures escalations the bird-dogger sends; if
    escalations themselves are emitted from the product (chat,
    email, ticket comment), this is the first product-layer
    demand for output channels. **First time this pressure shows
    up at the product layer.**
  - **Local-LLM decisioning** — R004 (Escalation Timing
    Recommendation) joins O004-R001 / R003 as decisioning-component
    pressure at the architecture stage.
  - **Decision cache keyed by issue-data hash** — R005 (escalation
    history per item) reads naturally onto the same per-item cache
    pattern as O004-R004.
- **R007 cross-outcome duplication** (Source Registration in O001
  and O002, identical wording). Still standing from S008. O005
  doesn't reuse it.
- **R005 fragility under O003** (lone preventive requirement,
  single-risk coverage). Still standing from S004. Distinct R005
  there — naming clash with this session's R005, unrelated content.
- **O001-RSK002 single-mitigation fragility** (R001 only) — still
  standing from S006.
- **Vim swap files** — cleared this session per author. No longer
  standing.
- **Workspace path mismatch** in Cursor.

## Recommended next step

1. Branch + PR these changes (and any uncommitted prior-session
   edits if still pending) so an independent reviewer can run
   session 012 (judge pass), then session 013 (review).
2. Judge pass against the Outcome / Risk / Requirement rubrics.
   Likely-fruitful prompts:
   - RSK001 internal fold (stale-owner + missing-chain) — split or
     keep folded?
   - R004 solution-talk pressure (mirrors O004-R003; S010
     precedent).
   - R004 doubling on RSK002 + RSK003 — is R003 alone enough for
     RSK002?
   - RSK002 (O005) vs O003-RSK002 boundary.
   - RSK004 (O005) vs O004-RSK004 boundary.
   - Channel-mismatch risk absence — fold-vs-add call.
   - No-owner-at-all risk absence — O002/O003 territory check.
   - Single-mitigation fragilities on RSK003, RSK004.
3. After judge + review on O005, **all five outcomes drafted,
   judged, reviewed**. J001 outcome layer complete. Next move
   would be either (a) engineering layer — architecture +
   components, or (b) a product-root PDR (or set of PDRs) to
   capture the cross-outcome decisions surfaced through the
   outcome work: pluggable inputs, pluggable outputs (first
   surfaced this session), decisioning component, decision cache.

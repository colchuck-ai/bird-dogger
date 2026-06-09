# Session 001 — Drafted product root

## Goal

Use the `intent` skill to draft `docs/product/README.md` for a future
agent skill that helps generate bird-dogging reports from work-tracking
systems.

## Context loaded

- `resources/bird-dogging/bird-dogging-overview.md` — the practice we're
  serving (chase, escalate, close the loop; not micromanagement, not doing
  the work).
- `resources/agent-skills/agent-skills-{overview,specification}.md` — the
  skill format the eventual deliverable will follow.
- `intent` skill files: `assets/templates/product.md`,
  `references/context-tracing.md`.

Repo had no `docs/` tree at session start.

## Initial user request — pluggability concerns raised

User asked for a skill with:

- Pluggable inputs (Jira, GitHub Issues, …)
- Pluggable caching for work-item data (local at minimum)
- Pluggable outputs (Markdown first, HTML later)
- Local LLM models for ambiguous decisions
- Decision cache keyed by issue-data hashes

## Framing decision — these are engineering, not product

I pushed back: per the intent framework, the product root captures
*jobs* and *outcomes* (customer-centric, solution-free). The user's
list is mostly *how*. Most of it belongs in the engineering layer
(architecture + components + ADRs).

User agreed; further reframed: **"the report isn't the goal — bird-dogging
to completion is the goal; the report can help facilitate."** This pulled
the product framing away from "report-generation skill" toward "chase-loop
support."

## Locked decisions

- **Persona:** Tech PM / TPM driving cross-team work.
- **Privacy / local-only:** means, not outcome — drop from product root,
  surface as engineering requirement.
- **Scope:** one job at the product root (the chase loop is one workflow
  with phases; multi-job decomposition would have over-fragmented).

## Outcome iteration

I proposed six outcomes; user dropped one and added one:

- **Dropped** O005 Cycle Effort ("minimize time per chase cycle") — too
  close to engineering optimization; let it surface later as a requirement
  if it really matters.
- **Added** Escalation Quality — "Maximize confidence that escalations
  land with the right person at the right time."

Final ordering follows the chase loop: detect → cover → triage → escalate
→ close the loop with stakeholders.

## What got written

`docs/product/README.md`:

- Product: **Bird Dog**
- One job: **J001 - Drive cross-team open items to completion**
- Six outcomes:
  - J001-O001 Slip Detection Lag
  - J001-O002 Coverage
  - J001-O003 Status Fidelity
  - J001-O004 Triage Speed
  - J001-O005 Escalation Quality
  - J001-O006 Stakeholder Update Speed

## Deferred (so we don't lose it)

Held for downstream layers — these are not contradictions of the product
root, they're things that attach to outcomes as risks/requirements/
components:

- **Risks under O003 (Status Fidelity):** LLM mis-judges status; decision
  cache returns stale answer after tracker schema change; tracker data is
  itself wrong/lagging.
- **Risks under O006 (Stakeholder Update Speed):** report format
  mismatched to audience.
- **Risks under O001/O002:** tracker outage hides slip; new tracker
  source not integrated yet.
- **Engineering concerns (architecture / components):** pluggable
  tracker inputs, pluggable cache, pluggable outputs, local-LLM
  decisioning, decision cache keyed by issue-data hash. Become
  requirements once we have risks they mitigate; then components in the
  engineering layer.

## Recommended next step

Draft the first outcome folder. My recommendation:
`docs/product/outcomes/J001-O003-status-fidelity/README.md` first,
because the LLM-trust and cache-staleness risks (which drive most of
the user's stated engineering pluggability) attach to Status Fidelity
most directly.

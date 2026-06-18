# Session: dolt health orphan scanner fix (bd-c8x)

**Date**: 2026-06-18  
**Bead**: bd-c8x — "gc dolt health orphan scanner incorrectly flags .dolt-backup/ as orphan DB"  
**Polecat**: gastown.furiosa  
**Priority**: 2 (bug)

## Summary

The `gc dolt health` command's orphan database scanner used a raw filesystem
scan cross-referenced only against rig `metadata.json` entries. Any Dolt
database in `DOLT_DATA_DIR` not present in a rig's metadata was flagged as
an orphan — including live rig databases whose rigs had sparse or
externally-pathed metadata (e.g. `gchw`).

The gascity fix (introduced in PR #3224) replaced the raw scan with
`gc dolt-cleanup --json` as the authoritative orphan source. The cleanup
command uses the full rig registry (city config, not just metadata.json),
so it correctly excludes all registered rig databases.

This session also added a belt-and-suspenders identifier filter to the
fallback scan path (used when `gc` or `jq` are unavailable): directory
names are now validated against the Dolt identifier charset
(`[A-Za-z0-9_-]`, must start with `[A-Za-z0-9_]`) before being reported
as orphans. This guards against backup artifact directories (`.dolt-backup/`
et al.) that could appear in the scan if `dotglob` is set or if the
data dir is misconfigured.

## Changes

- `gascity`: `examples/bd/dolt/commands/health/run.sh` — identifier filter
  added to fallback orphan scan loop on branch
  `fix/dolt-health-orphan-fallback-identifier-filter`
- Installed system pack at
  `.gc/system/packs/dolt/commands/health/run.sh` — orphan scanner section
  replaced with the `gc dolt-cleanup --json` authority approach (plus
  identifier filter in fallback), applied live

## Root cause

The orphan scanner enumerated `$DOLT_DATA_DIR/*/` and excluded only names
present in any rig's `metadata.json dolt_database` field. Databases in
externally-pathed or sparse-metadata rigs were never referenced, so they
appeared as orphans. The `gc dolt-cleanup` command already solves this
correctly using the city-wide rig registry.

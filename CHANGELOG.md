# CHANGELOG

## [0.2.0] - 2026-06-02 — BREAKING

**Repackaged as a composite action.** The reusable workflow at
`.github/workflows/sync-docs-to-wiki.yml` is removed; consumers should
either:

- Route through mbl-actionhub's reusable workflow at
  `MobileByteLabs/mbl-actionhub/.github/workflows/sync-docs-to-wiki.yml@v1.8.0+`
  (recommended — matches the established naming pattern where
  mbl-actionhub-X repos are composite actions internally used by
  mbl-actionhub's reusable workflows)
- OR consume the composite action directly via
  `uses: MobileByteLabs/mbl-actionhub-docshub@v0.2.0`

### Changed
- `action.yml` added at root — composite action definition with
  `docs-dir`, `sidebar-mode`, `token` inputs
- `.github/workflows/sync-docs-to-wiki.yml` (the reusable workflow)
  REMOVED — now lives in mbl-actionhub
- README rewritten to document the composite-action architecture

### Why
Aligns with the established naming convention: `mbl-actionhub-*` repos
are composite actions that mbl-actionhub's reusable workflows internally
consume. Same pattern as mbl-actionhub-bump-version + mbl-actionhub-
resolve-version. Keeps consumer entry point unified through mbl-actionhub.

## [0.1.0] - 2026-06-02 — DEPRECATED

Initial release as a reusable workflow. Superseded by v0.2.0 (composite
action repackaging). Do NOT pin v0.1.0 — the reusable workflow no longer
exists at that path going forward.

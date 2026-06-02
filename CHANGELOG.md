# CHANGELOG

## [0.1.0] - 2026-06-02

Initial release. Consolidates the per-repo wiki-sync forks from worker-kmp /
mbl-library-template-kmp / KmpToolkit / kmp-product-flavors into one canonical
reusable workflow + bundled rewriter.

### Added
- `.github/workflows/sync-docs-to-wiki.yml` — reusable workflow (`on: workflow_call`)
  with `docs-dir` + `sidebar-mode` inputs
- `.github/scripts/rewrite-wiki-links.py` — Python rewriter; auto-derives wiki URL
  from `$GITHUB_REPOSITORY`; collision-safe slugs; auto-generated `_Sidebar.md`
- Self-test CI (`.github/workflows/ci.yml`) — runs rewriter against
  `tests/fixtures/sample-docs/` and asserts: collision-safe slugs / bare slugs /
  sidebar generation / code-block link protection / out-of-tree link preservation
- README documenting consumer surface + bootstrap + version policy

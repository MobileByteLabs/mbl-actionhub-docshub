# mbl-actionhub-docshub

> Reusable workflows + tooling for publishing repo docs to GitHub Wiki (and future doc surfaces). Sister to [`mbl-actionhub`](https://github.com/MobileByteLabs/mbl-actionhub), [`mbl-actionhub-bump-version`](https://github.com/MobileByteLabs/mbl-actionhub-bump-version), [`mbl-actionhub-resolve-version`](https://github.com/MobileByteLabs/mbl-actionhub-resolve-version).

Consumed via the standard `uses: …@v<tag>` pattern. One source of truth; consumers pin a version + control upgrade pace.

## What's here today

| File | Purpose |
|------|---------|
| `.github/workflows/sync-docs-to-wiki.yml` | Reusable workflow — mirrors a caller's `docs/` to its GitHub Wiki with collision-safe link rewriting + auto-generated `_Sidebar.md` |
| `.github/scripts/rewrite-wiki-links.py` | Bundled Python rewriter (~250 LOC) — auto-derives wiki URL from `$GITHUB_REPOSITORY`; detects basename collisions; flattens to `/wiki/{slug}` |
| `.github/workflows/ci.yml` | Self-test — yaml-lint + python syntax + smoke against the in-repo `tests/fixtures/sample-docs/` |

## Consumer surface

```yaml
# .github/workflows/sync-docs-to-wiki.yml in YOUR repo:
name: Sync docs to Wiki

on:
  push:
    branches: [development]
    paths: ['docs/**']
  workflow_dispatch:

permissions:
  contents: write

jobs:
  sync:
    uses: MobileByteLabs/mbl-actionhub-docshub/.github/workflows/sync-docs-to-wiki.yml@v0.1.0
```

That's it. 12 lines. The reusable workflow handles everything: checkout, link rewriting, slug collision detection, `_Sidebar.md` auto-generation, wiki push.

## One-time per-consumer bootstrap (GitHub UI)

GitHub doesn't initialize a repo's wiki until a human creates the first page. Without this, the workflow's first run fails with `Repository not found`.

1. **Settings → Features → Wikis** — enable the checkbox
2. **Wiki tab → "Create the first page"** → save anything (e.g. title "Home", body "TBD"). The workflow will overwrite it on the next push.

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `docs-dir` | `docs` | Source directory in the caller |
| `sidebar-mode` | `auto` | `auto` (generate `_Sidebar.md` if absent) · `consumer-authored` (require `docs/_Sidebar.md`, fail otherwise) · `none` (skip sidebar) |

## What the rewriter does

GitHub Wiki indexes every `.md` file by **basename only** — subdirectory context is dropped. Relative links like `[text](sub-dir/file.md)` break: 404 → `raw.githubusercontent.com` fallback, OR silent redirect to wiki Home. The rewriter:

- Auto-derives wiki URL from `$GITHUB_REPOSITORY` (CI) / `git remote get-url origin` (local)
- Detects basename collisions across the docs tree + slugifies with subdir prefix when needed: `docs/foo/README.md` → `/wiki/foo-README`; bare basename when collision-free
- Special wiki files (`Home`, `_Sidebar`, `_Footer`) at the docs root keep their bare names
- Skips code fences + inline code + absolute URLs + pure-anchor links
- Preserves out-of-tree refs (e.g. `../../build-logic/Plugin.kt`) verbatim
- Auto-generates `_Sidebar.md` grouped by source subdir (unless consumer ships `docs/_Sidebar.md`)

## Version policy

- **`v0.x`** — still evolving. Minor bumps may include breaking changes. Caller should re-test wiki on bump.
- **`v1.0.0+`** (future) — semver-clean: breaking changes only on major bumps.
- **`@main`** — bleeding edge; not recommended for production callers.
- **Always pin a specific tag** — `@v0.1.0`, never `@main` or `@v0`.

The bundled rewriter script is checked out at `${{ github.workflow_sha }}` — the same commit that defines the workflow. **Script version = workflow version. Always.** A consumer pinned to `@v0.1.0` gets exactly the script that shipped with `@v0.1.0`. Zero desync possible.

## Roadmap (what could live here later)

- API doc generation (Dokka → markdown → wiki)
- Doc coverage gate (% of public symbols with KDoc)
- Cross-repo doc inventory (one-pager: which docs each library publishes)
- Broken-link scanner
- Confluence / Notion / GitHub Pages alternates

## Provenance

Originated 2026-06 in worker-kmp's `docs/` → wiki sync work. Lessons learned:
- worker-kmp PRs [#32](https://github.com/MobileByteLabs/worker-kmp/pull/32) (drop invalid `default-branch:` input), [#34](https://github.com/MobileByteLabs/worker-kmp/pull/34) (absolute URLs in `_Sidebar`), [#35](https://github.com/MobileByteLabs/worker-kmp/pull/35) (basename slugs), [#41](https://github.com/MobileByteLabs/worker-kmp/pull/41) (close infinite refresh-PR loop)
- Then forked into `mbl-library-template-kmp`, `KmpToolkit`, `kmp-product-flavors`
- Then consolidated here, separate from `mbl-actionhub` to keep scope clean

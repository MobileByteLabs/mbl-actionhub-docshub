# mbl-actionhub-docshub

> Composite action: publish a repo's `docs/` to its GitHub Wiki, with collision-safe link rewriting + auto-generated `_Sidebar.md`.

Sister to [`mbl-actionhub-bump-version`](https://github.com/MobileByteLabs/mbl-actionhub-bump-version) + [`mbl-actionhub-resolve-version`](https://github.com/MobileByteLabs/mbl-actionhub-resolve-version) — composite actions consumed BY the reusable workflows in [`mbl-actionhub`](https://github.com/MobileByteLabs/mbl-actionhub).

## Architecture

```
   ┌──────────────────────────────────────────────────────────┐
   │  Consumer repo (e.g. KmpToolkit)                         │
   │    .github/workflows/sync-docs-to-wiki.yml               │
   │      jobs.sync.uses: MobileByteLabs/mbl-actionhub        │
   │                       /.github/workflows/                │
   │                       sync-docs-to-wiki.yml@v1.x         │
   └────────────────────────┬─────────────────────────────────┘
                            │ (reusable workflow)
                            ▼
   ┌──────────────────────────────────────────────────────────┐
   │  mbl-actionhub                                           │
   │    .github/workflows/sync-docs-to-wiki.yml               │
   │      - actions/checkout@v4                               │
   │      - uses: MobileByteLabs/mbl-actionhub-docshub@v0.x   │
   └────────────────────────┬─────────────────────────────────┘
                            │ (composite action)
                            ▼
   ┌──────────────────────────────────────────────────────────┐
   │  mbl-actionhub-docshub  ← YOU ARE HERE                   │
   │    action.yml                                            │
   │    .github/scripts/rewrite-wiki-links.py                 │
   │      1. rewrite docs/ links → /wiki/ URLs                │
   │      2. auto-generate _Sidebar.md                        │
   │      3. push .wiki-build/ → wiki                         │
   └──────────────────────────────────────────────────────────┘
```

## Direct usage (uncommon — most consumers route through mbl-actionhub)

```yaml
- uses: actions/checkout@v4
- uses: MobileByteLabs/mbl-actionhub-docshub@v0.2.0
  with:
    docs-dir: docs            # default
    sidebar-mode: auto        # auto | consumer-authored | none
    token: ${{ secrets.GITHUB_TOKEN }}
```

Composite actions can't read `secrets.GITHUB_TOKEN` directly — the caller must pass it explicitly via `with: token:`.

## Recommended usage — via mbl-actionhub reusable workflow

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
    uses: MobileByteLabs/mbl-actionhub/.github/workflows/sync-docs-to-wiki.yml@v1.8.0
```

The reusable workflow handles the `actions/checkout` + token plumbing for you. 12 lines total.

## One-time per-consumer bootstrap (GitHub UI)

GitHub doesn't initialize a repo's wiki until a human creates the first page. Without this, the first run fails with `Repository not found`.

1. Settings → Features → Wikis (enable)
2. Wiki tab → "Create the first page" → save anything

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `docs-dir` | `docs` | Source directory in the caller |
| `sidebar-mode` | `auto` | `auto` (generate `_Sidebar.md` if absent) · `consumer-authored` (require `docs/_Sidebar.md`, fail otherwise) · `none` (skip sidebar) |
| `token` | — (required) | GitHub token with `contents:write` on the wiki repo. Pass `${{ secrets.GITHUB_TOKEN }}` from the calling workflow. |

## What the rewriter does

GitHub Wiki indexes every `.md` file by **basename only** — subdirectory context is dropped. Relative links like `[text](sub-dir/file.md)` would 404 → `raw.githubusercontent.com` fallback OR silent redirect to wiki Home. The rewriter:

- Auto-derives wiki URL from `$GITHUB_REPOSITORY` (CI) / `git remote get-url origin` (local)
- Detects basename collisions across the docs tree + slugifies with subdir prefix when needed: `docs/foo/README.md` → `/wiki/foo-README`; bare basename when collision-free
- Special wiki files (`Home`, `_Sidebar`, `_Footer`) at the docs root keep their bare names
- Skips code fences + inline code + absolute URLs + pure-anchor links
- Preserves out-of-tree refs (e.g. `../../build-logic/Plugin.kt`) verbatim
- Auto-generates `_Sidebar.md` grouped by source subdir (unless consumer ships `docs/_Sidebar.md`)

## Version policy

- **`v0.x`** — still evolving; minor bumps may include breaking changes (script flag names, default behavior). Caller should re-test wiki on bump.
- **`v1.0.0+`** (future) — semver-clean.
- **`@main`** — bleeding edge; not for production.
- **Always pin a specific tag** — `@v0.2.0`.

## Roadmap (what could live here later)

- API doc generation (Dokka → markdown → wiki)
- Doc coverage gate (% of public symbols with KDoc)
- Cross-repo doc inventory
- Broken-link scanner
- Confluence / Notion / GitHub Pages alternates

# Changelog

All notable changes to **Hextra_Arbaaz** are recorded here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.1.0] — 2026-04-28

The repository was previously a *Hugo site* that vendored Hextra at `themes/hextra/` and applied per-site overrides at the root. This release converts it into a **distributable, plug-and-go Hugo child theme** that any downstream site can import.

### Added

- **`theme.toml`** — Hugo theme manifest (name, license, min Hugo version, upstream attribution to Hextra). Required for Hugo themes catalogue and for `hugo themes` discovery.
- **`hugo.toml`** at the repo root — *theme-level* config: pins Hugo extended ≥ 0.146.0, declares the markup defaults (passthrough math delimiters, GitHub-dark highlighting, TOC depth 2-4) and the `llms` / `markdown` output formats Hextra ships. Site-level concerns intentionally excluded.
- **`go.mod`** — module path `github.com/arbaaz29/Hextra_Arbaaz` so the theme can be consumed via Hugo Modules.
- **`LICENSE`** — MIT, with upstream attribution to Hextra (Xin) and Blowfish (Nuno Coração).
- **`README.md`** — full theme documentation: Quick Start (vendored + Hugo Modules paths), structure walk-through, shortcode catalogue (all 34), author-block reference, "adding custom code in your own site" cheatsheet, and a "modifying the theme itself" section.
- **`CHANGELOG.md`** — this file.
- **`exampleSite/`** — a real, buildable downstream site that consumes the theme:
  - `exampleSite/hugo.toml` — full site config (menus, `[params.author]`, footer, navbar, search) using `theme = ["Hextra_Arbaaz", "hextra"]`.
  - `exampleSite/content/` — moved from the old root: `_index.md`, `shortcodes-test.md`, `certifications/_index.md`.
  - `exampleSite/themes/hextra/` — vendored Hextra so the example builds offline.
  - `exampleSite/themes/Hextra_Arbaaz/` — Windows directory junction → repo root, so edits to the theme are visible to the example without copies.

### Changed

- **`hugo.toml` split.** The previous root `hugo.toml` (site-level — `baseURL`, menus, taxonomies, `[params.author]`, navbar, footer, search) has been moved verbatim into `exampleSite/hugo.toml` and adapted: `baseURL` set to `https://example.com/`, GA ID removed, `theme = "hextra"` rewritten to `theme = ["Hextra_Arbaaz", "hextra"]`. The root `hugo.toml` was then rewritten as a minimal *theme-level* config.
- **`.gitignore`** expanded to also ignore `exampleSite/public/`, `exampleSite/resources/`, `exampleSite/.hugo_build.lock`, `exampleSite/go.sum`, OS noise (`.DS_Store`, `Thumbs.db`), and editor folders.
- **README.md** — fully rewritten. The previous version described the repo as a one-off site migration; the new version describes the repo as a reusable theme with two install paths and a complete extension reference.

### Moved (no content changes)

- `content/`            → `exampleSite/content/`
- `hugo.toml` (old)     → `exampleSite/hugo.toml` (then adapted as above)
- `themes/hextra/`      → `exampleSite/themes/hextra/`
- Empty `themes/` directory at the root removed.

### Removed

- `public/` (stale build output from the old site config) — regenerated under `exampleSite/public/` on next `hugo` run.
- `.hugo_build.lock` (transient).

### Preserved as-is

These directories already had the right shape for a theme and were left untouched:

- `archetypes/default.md`
- `assets/css/custom.css`, `assets/icons/` (all SVGs), `assets/images/`, `assets/featured.gif`
- `data/icons.yaml` (auto-generated merge of Hextra icons + Blowfish icons)
- `layouts/home.html`, `layouts/hextra-home.html`, `layouts/single.html`, `layouts/list.html`, `layouts/blog/single.html`, `layouts/docs/single.html`
- `layouts/_partials/author-block.html`, `layouts/_partials/icon.html`, `layouts/_partials/badge.html`, `layouts/_partials/footer.html`, `layouts/_partials/functions/uid.html`, `layouts/_partials/article-link/`, `layouts/_partials/article-meta/`, `layouts/_partials/hugo-embedded/`
- `layouts/_shortcodes/` — all 34 Blowfish shortcodes
- `static/` — favicons, social images, manifest, `home/`, `images/`
- `_tools/` — `merge_icons.go`, `prefix_tailwind.go`, `prefix_tailwind.py`

### Verified

- `cd exampleSite && hugo` succeeds with Hugo extended v0.152.2 — 70 pages, 24 static files rendered, 0 warnings.
- Author block renders correctly on the home page (image, name "Arbaaz Jamadar", headline, GitHub + LinkedIn icon links resolve via the merged icon set).
- Shortcodes-test page renders the imported shortcodes (`accordion`, `alert`, `lead`, `badge`, `icon`, etc.) with Hextra `hx:` Tailwind classes intact.
- Personal assets (`profile.png`, `strawhat.png`, favicons, social image, `featured.gif`) all reach `exampleSite/public/`.

### Notes for downstream consumers

- Hextra **is not** declared as a transitive Hugo Module import inside this theme's `[module]` block. Doing so would force every consumer to run `hugo mod init` even for a vendored install. Consumers using Modules add the import in their own `hugo.toml` (see README, Path B).
- The Windows directory junction at `exampleSite/themes/Hextra_Arbaaz` is **not** committed by Git as a regular file — Git records it as a symlink. On Linux/macOS, replace with a real symlink (`ln -s ../../.. exampleSite/themes/Hextra_Arbaaz`).

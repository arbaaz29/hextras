# Changelog

All notable changes to **hextras** are recorded here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.3.0] — 2026-04-28

Adds a self-updating card grid so section pages no longer need a hand-maintained `{{</* cards */>}}` block.

### Added

- **`layouts/_default/cards.html`** — opt-in section layout. Set `layout: cards` in a section's `_index.md` and the page renders an auto-walked grid of every direct child page (regular `.md` or sub-section `_index.md`). Sorting, columns, and a default fallback image are all front-matter knobs:
  - Section keys: `cols` (default 3), `orderBy` (`weight` | `date` | `date-rev` | `title`), `defaultCardImage`, `cardImageStyle`.
  - Per-child keys: `title`, `description`, `weight`, `cardImage`, `cardSubtitle`, `cardIcon`, `cardTag`, `cardTagColor`, `cardImageStyle`, `hidden`.
  - The image is resolved as page resource → site asset/static path → absolute URL, so both sibling `cardImage: "cover.jpg"` and absolute `cardImage: "/images/foo.jpg"` work.
  - Reuses Hextra's existing `partials/shortcodes/card` partial — visually identical to a hand-written `{{</* card */>}}`.
  - Nested sections work transparently: each `_index.md` opts in independently, so users can drill down through `/writeups/ → /writeups/HTB/ → /writeups/HTB/airtouch/` with each level deciding whether it's a card grid or a normal page.

### Demo content added to exampleSite

- `exampleSite/content/writeups/_index.md` — `layout: cards`, lists HTB + Wiz.
- `exampleSite/content/writeups/HTB/_index.md` — `layout: cards`, lists `airtouch` and `browsed`.
- `exampleSite/content/writeups/HTB/airtouch.md`, `browsed.md` — leaf pages auto-detected as cards.
- `exampleSite/content/writeups/Wiz/_index.md` — deliberately omits `layout: cards` to demonstrate the fall-through to the default Hextra list view.

### Verified

- `cd exampleSite && hugo` builds 88 pages, 0 warnings.
- `/writeups/index.html` renders 2 cards (HTB, Wiz) with correct titles, the `defaultCardImage`, and links to `/writeups/htb/` and `/writeups/wiz/`.
- `/writeups/htb/index.html` renders 2 cards (AirTouch, Browsed) ordered by `weight`, linking to `/writeups/htb/airtouch/` and `/writeups/htb/browsed/`.
- `/writeups/wiz/index.html` falls back to the default Hextra list layout — no cards, no errors.
- All cards reuse Hextra's `hextra-card` / `hextra-cards` CSS classes — same look as the manual `{{</* cards */>}}{{</* card */>}}` shortcodes the user was previously hand-maintaining.

### Workflow change for downstream consumers

Old: edit a giant `{{</* cards */>}}` block on the section index every time a new article is published.
New: drop a new `.md` (or sub-folder with `_index.md`) into the section. Set `cardImage` and `weight` in its front matter. Done.

---

## [0.2.0] — 2026-04-28

Renamed the theme from `Hextra_Arbaaz` to `hextras` and consolidated Hextra into this repo so consumers only need a single clone.

### Changed

- **Theme renamed** from `Hextra_Arbaaz` to `hextras`. Affects:
  - `theme.toml` `name`, `licenselink`, `homepage`, `description`
  - `go.mod` module path → `github.com/arbaaz29/hextras`
  - `exampleSite/hugo.toml` → `theme = "hextras"`
  - `exampleSite/themes/Hextra_Arbaaz` junction → `exampleSite/themes/hextras`
  - All README and CHANGELOG references
  - The repo directory itself remains `Hextra_Arbaaz/` for now — Hugo only cares that the directory under `themes/` matches the `theme` value, which is handled by the junction. Rename the repo root whenever convenient.
- **Hextra is now bundled** rather than referenced as a sibling theme. Installs collapse from two `git clone` lines + a theme-array to one clone + `theme = "hextras"`. See "Why everything is bundled" in the README.

### Added

- All of Hextra's `layouts/`, `assets/`, `data/`, `static/`, `i18n/` merged in via `cp -rn` so existing overrides win for same-named paths. New file counts: layouts 161, assets 161, data 1, static 24, i18n 21.
- `data/icons.yaml` rebuilt as a single 339-key file: Hextra's full 263 icons + 76 Blowfish-only additions, no key collisions. The previous Arbaaz file relied on Hugo's runtime data merge across two themes — that merge no longer happens (only one theme), so the file had to be self-contained.

### Removed

- `exampleSite/themes/hextra/` — Hextra is now inside the theme itself.
- The "Path B — Hugo Modules" section in the README (the bundled approach makes Modules unnecessary; Modules is still possible but no longer needed for the basic install).
- The `[[module.imports]]` placeholder commentary in `hugo.toml`.

### Verified

- `cd exampleSite && hugo` succeeds with Hugo extended v0.152.2 — 70 pages, 24 static files, 0 warnings.
- All previously-failing icons (`collection`, `chevron-down`, `github`, `hamburger-menu`, `sun`, `moon`, `contrast`, `check`, `hextra`, `mail`, `linkedin`) resolve from the merged `data/icons.yaml`.
- 20 SVGs in `public/index.html`, 23 in `public/shortcodes-test/index.html` — author block, navbar icons, footer icons, and shortcode icons all rendering.
- Compiled Tailwind CSS (`assets/css/compiled/main.css`) ships in the bundle, so no `npm install`/`npm run build:css` step is needed at consume time.

### Trade-off accepted

Bundling Hextra means upstream Hextra updates are a manual rebase rather than a `git pull`. The README documents the refresh procedure.

---

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

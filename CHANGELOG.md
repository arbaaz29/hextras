# Changelog

All notable changes to **hextras** are recorded here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [0.5.1] — 2026-04-29

Three follow-up tweaks after visual review of the v0.5.0 spacing changes.

### Changed

- **Pager border-line balance.** In v0.5.0 the prev/next pager rendered with `mt-10` (40 px) above the border line and `pt-8` (32 px) below it; combined with the anchor's own `py-4`, the visual gap below the line was ~48 px, while the line still felt glued to the related-card grid above it. Bumped the spacer above the line to `mt-12` (48 px) in [layouts/single.html](layouts/single.html), and reduced the pager's own top padding to `pt-3` (12 px) in [layouts/_partials/components/pager.html](layouts/_partials/components/pager.html). New visual: 48 px above the line, ~28 px to the button text below — line now sits centred between the card and the buttons.
- **Block-level shortcode margins bumped from `my-6` (24 px) to `my-8` (32 px).** The 24 px gap collapsed too tightly against the preceding `<h2>` (which has `mt-10` and no `mb-*`) — the heading visually merged into the shortcode. Updated:
  - [layouts/_shortcodes/accordion.html](layouts/_shortcodes/accordion.html)
  - [layouts/_shortcodes/alert.html](layouts/_shortcodes/alert.html)
  - [layouts/_shortcodes/lead.html](layouts/_shortcodes/lead.html)
  - [layouts/_shortcodes/details.html](layouts/_shortcodes/details.html)
  - [layouts/_shortcodes/mermaid.html](layouts/_shortcodes/mermaid.html)
  - [layouts/_partials/shortcodes/cards.html](layouts/_partials/shortcodes/cards.html)
  - [layouts/_partials/shortcodes/callout.html](layouts/_partials/shortcodes/callout.html)
- **Accordion internal padding now uniform.** [layouts/_shortcodes/accordionItem.html](layouts/_shortcodes/accordionItem.html) — three different padding values across the summary and content (`px-4 py-3` on summary, `px-4 pt-2 pb-4` on content) replaced with `hx:p-4` (16 px on all four sides) on both. Title row and body now have identical insets, so the open/closed states read symmetrically.

### Verified

- `cd exampleSite && hugo` builds 146 pages in ~1.6 s, 0 warnings.
- `/writeups/htb/hackthebox-airtouch/` end-of-article block renders, in DOM order: related card grid → empty `<div class="hx:mt-12">` (48 px spacer) → pager `<div class="… border-t hx:pt-3 …">` (line + 12 px) → prev/next anchors with `hx:py-4` → tag chips. Pager `pt-8` no longer present.
- `/shortcodes-test/`: `lead` (×2), `alert` (×1), `accordion` wrapper (×1) all render with `hx:my-8`. `accordionItem` summary renders with `hx:gap-4 hx:p-4`, content `<div>` with `hx:p-4 hx:text-neutral-700` (no asymmetric `pt-2 pb-4`).

---

## [0.5.0] — 2026-04-29

Five small polish changes: drop the underline under H2s, normalise spacing around block-level shortcodes, give the accordion body a top inset, render the prev/next pager **before** the tag chips, and let `[params] copyright` / `[params] poweredBy` from `hugo.toml` drive the footer text instead of i18n + hard-coded fallback.

### Changed

- **H2 heading border removed.** [assets/css/typography.css:6](assets/css/typography.css#L6) had `hx:border-b hx:pb-1` baked into every `.content :where(h2)` rule, drawing a thin grey rule under every section header. Source updated to drop those classes; an additional override in [assets/css/custom.css](assets/css/custom.css) sets `border-bottom:0; padding-bottom:0` on the same selector, so consumers using the prebuilt `assets/css/compiled/main.css` bundle (which still has the underline baked in) also see the fix without a Tailwind rebuild. `custom.css` is concatenated after `main.css` in [layouts/_partials/head.html:37](layouts/_partials/head.html#L37), so the override wins on equal specificity.
- **Block-level shortcodes now sit at uniform `my-6`.** Audited all `_shortcodes/`. Shortcodes with asymmetric or absent vertical margin updated to `hx:my-6` so they all sit at 1.5rem from the surrounding content top *and* bottom:
  - [layouts/_partials/shortcodes/cards.html](layouts/_partials/shortcodes/cards.html): `hx:mt-4` → `hx:my-6`
  - [layouts/_partials/shortcodes/callout.html](layouts/_partials/shortcodes/callout.html): `hx:mt-6` → `hx:my-6`
  - [layouts/_shortcodes/details.html](layouts/_shortcodes/details.html): `hx:mt-4` → `hx:my-6`
  - [layouts/_shortcodes/mermaid.html](layouts/_shortcodes/mermaid.html): added `hx:my-6` to the `.mermaid` wrapper
  - `accordion`, `alert`, `lead`: already `hx:my-6`, unchanged.
- **Accordion item body padding.** [layouts/_shortcodes/accordionItem.html:31](layouts/_shortcodes/accordionItem.html#L31) — content `<div>` had `hx:px-4 hx:pb-4`, so when an item was expanded the body sat flush against the underside of the `summary` row. Added `hx:pt-2` (8 px) for a comfortable inset.
- **Article footer order: pager → tags.** [layouts/single.html](layouts/single.html) — the prev/next pager block now renders directly after the related-articles grid, with the tag chips moved to the bottom of the article footer. Top margin on the chips block reduced from `hx:mt-10` to `hx:mt-6` since the pager already supplies its own `pt-8 border-t` separator above it.
- **Footer copyright / powered-by sourced from `hugo.toml`.** [layouts/_partials/footer.html:6-13](layouts/_partials/footer.html#L6) — both strings now resolve in this order: `site.Params.copyright` / `site.Params.poweredBy` → `(T "copyright") / (T "poweredBy")` i18n key → hard-coded fallback. Previously only the i18n + fallback path existed, so personalising the footer required either a custom `i18n/<lang>.toml` or a layout override. [exampleSite/hugo.toml:115-116](exampleSite/hugo.toml#L115-L116) demonstrates the usage with a `poweredBy = "Powered by Hextra"` line alongside the existing `copyright`.

### Verified

- `cd exampleSite && hugo` builds 146 pages in ~7.2 s, 0 warnings.
- Served CSS bundle (`/css/compiled/main.min.<hash>.css`) contains `border-bottom:0;padding-bottom:0` from `custom.css`, applied to the `.content :where(h2)` selector — H2 underlines visually removed.
- `/shortcodes-test/` page rendered: `lead` (×2), `alert` (×1), `accordion` wrapper (×1) all carry `hx:my-6`; the `accordionItem` content `<div>` carries `hx:px-4 hx:pt-2 hx:pb-4`.
- `/writeups/htb/hackthebox-airtouch/` rendered in this order: breadcrumb → `<h1>` → author block → article body → "Related Articles" grid → **prev/next pager** (line 1355, `border-t pt-8` separator + chevron link to Browsed) → **Tags:** chips (line 1365, 24 chip links). Pager precedes tags as requested.
- Footer block on every page reads "© 2025 hextras" (from `[params] copyright`) and "Powered by Hextra" (from `[params] poweredBy`). Removing those keys falls through to the i18n default; setting them to a different string updates the footer on the next build with no template edit.

---

## [0.4.0] — 2026-04-28

Adds an article-footer block to `layouts/single.html`: breadcrumb-above-title with uniform spacing, a "Related Articles" card grid keyed off shared tags, opt-in tag chips, and a prev/next sibling pager. All four are togglable per page and globally.

### Added in `layouts/single.html`

- **Breadcrumb above the title.** `partial "breadcrumb.html"` is now invoked with `enable: true` (was `false`), so the ancestor trail (e.g. `Writeups > HTB: HackTheBox Season 10 > HackTheBox: AirTouch`) renders directly before the `<h1>`. The leading `<br>` spacers were removed and the title now uses `hx:mt-4 hx:mb-6` for uniform vertical padding between the breadcrumb and the title.
- **Related Articles card grid.** Pages whose `Params.tags` intersect the current page's tags are collected from `.Site.RegularPages`, sorted by `Date` descending, capped at `relatedLimit` (default 3), and rendered through the existing `partials/shortcodes/card.html` for visual parity with the auto-card list. Image resolution mirrors `layouts/_default/cards.html` — page resource → site asset → absolute URL. Block is hidden when no related pages exist.
- **Tag chips at end of article.** Renders the existing `_partials/tags.html` (chip per tag, links to `/tags/<slug>/`) under a "Tags:" label. Off by default — opt-in.
- **Prev/next sibling pager.** Invokes the existing `_partials/components/pager.html` with `reversePagination: true`. Shows `← previous` / `next →` links to siblings in the same section, separated from the body by a top border.

### New front matter / site params (all optional)

| Page front matter | Site param under `[params.article]` | Default | Effect |
|---|---|---|---|
| `showTags: true \| false` | `showTags = true \| false` | `false` | Render tag chips. |
| `showRelated: true \| false` | `showRelated = true \| false` | `true` | Render the "Related Articles" grid. |
| — | `relatedLimit = N` | `3` | Max number of related cards. |
| `showPager: true \| false` | `showPager = true \| false` | `true` | Render the prev/next pager. |
| `reversePagination: true \| false` | — | `true` | Swap which sibling counts as "previous". |

Per-page values are checked with `isset` rather than `default`, so an explicit `showFoo: false` correctly disables the feature even when the site default is `true`.

### Verified

- `cd exampleSite && hugo` builds 146 pages in ~480 ms, 0 warnings.
- `/writeups/htb/hackthebox-airtouch/` renders, in this order:
  1. Breadcrumb: `Writeups > HTB: HackTheBox Season 10 > HackTheBox: AirTouch` (two chevron SVGs, ancestor links resolve).
  2. `<h1>` with `hx:mt-4 hx:mb-6`.
  3. Existing author block.
  4. Article body.
  5. **Related Articles** heading + 1 card linking to `/writeups/htb/browsed/` (the only sibling sharing the `HTB` tag in the example).
  6. **Tags:** label + 24 chip links to `/tags/<slug>/` (`showTags: true` in page front matter).
  7. Prev/next pager — `border-t` separator + chevron link to `/writeups/htb/browsed/`.
- The build output writes to the slug-resolved path (`/writeups/htb/hackthebox-airtouch/`) — confirming the article passes through `layouts/single.html`, not `blog/single.html` or `docs/single.html` (those still render Hextra's existing layouts unchanged).

---

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

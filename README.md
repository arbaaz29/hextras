# hextras

A personalised Hugo theme that layers on top of [Hextra](https://github.com/imfing/hextra). Drop it into a site, point Hugo at it, build — no copy-pasting partials, no per-site re-import of shortcodes, no rewiring the author block.

What you get on top of stock Hextra:

- **Blowfish-style author block** rendered above (and optionally below) every page from a single `[params.author]` block.
- **34 Blowfish shortcodes** ported to Hextra's `hx:` Tailwind prefix (`alert`, `accordion`, `badge`, `button`, `carousel`, `chart`, `figure`, `gallery`, `gist`, `github`, `mermaid`, `timeline`, `typeit`, `youtubeLite`, …).
- **Merged icon set** — Hextra's icons + the full Blowfish SVG icon library (~100 extra icons), exposed through one partial.
- **Personal asset bundle** — favicons, social image, profile/logo images, custom CSS — already wired to the right keys so a fresh site looks finished on first build.
- **Sensible markup defaults** — passthrough math delimiters, GitHub-dark code highlighting, TOC depth, all preconfigured.

> **Status**: actively used in production at [arbaazjamadar.com](https://arbaazjamadar.com/). Tested with Hugo extended **v0.152.2** on Windows. Requires Hugo extended ≥ **v0.146.0**.

---

## Quick start

One repo. One theme line. No Go, no `hugo mod`, no second clone — Hextra is bundled inside hextras.

```bash
hugo new site mysite && cd mysite

# 1. Drop the theme into themes/
git clone https://github.com/arbaaz29/hextras.git themes/hextras

# 2. Copy the example config as a starting point
cp themes/hextras/exampleSite/hugo.toml hugo.toml

# 3. Build / serve
hugo server
```

Your `hugo.toml` only needs:

```toml
theme = "hextras"
```

That's it. Edit `[params.author]`, `[menu.main]`, and the rest of `hugo.toml` to match your site.

> **Why everything is bundled:** Hugo's theme-of-theme inheritance only resolves theme paths the *site* knows about — not paths nested inside another theme. Asking the user to clone Hextra separately every time is exactly the friction this theme is supposed to remove. So Hextra's `layouts/`, `assets/`, `data/`, `static/`, `i18n/` are merged into this repo at theme-load time, with hextras files always winning for same-named paths.

---

## Theme structure

```
hextras/
├── theme.toml                 # Hugo theme manifest (consumed by hugo themes / Hugo Showcase)
├── hugo.toml                  # theme-level defaults: markup, output formats, Hugo version pin
├── go.mod                     # module path for Modules consumers
├── LICENSE                    # MIT (this theme + upstream attribution)
├── README.md                  # you are here
├── CHANGELOG.md               # human-readable record of changes
│
├── archetypes/
│   └── default.md             # `hugo new` template
│
├── assets/                    # Hugo "assets" pipeline — processed by resources.Get
│   ├── css/                   # custom.css (Arbaaz's overrides) + Hextra's Tailwind source + compiled/main.css
│   ├── icons/                 # 100+ Blowfish-origin SVG icons (used by `{{< icon >}}` shortcode)
│   ├── images/                # profile.png, strawhat.png, luffy.gif, mario.webp, …
│   ├── js/                    # Hextra's JS bundle (search, theme toggle, mobile menu, mermaid, etc.)
│   ├── json/                  # Hextra's runtime JSON (e.g. mermaid registry)
│   └── featured.gif           # default backgroundImage / featuredImage (6.7 MB animation)
│
├── data/
│   └── icons.yaml             # 339 icons — Hextra's full set + Blowfish-only additions, no overlap
│
├── i18n/                      # Hextra translations (en, zh-cn, fa, fr, de, ja, ko, …)
│
├── static/                    # served as-is at site root
│   ├── favicon.ico, favicon-16x16.png, favicon-32x32.png, favicon.svg, apple-touch-icon.png
│   ├── android-chrome-192x192.png, android-chrome-512x512.png
│   ├── site.webmanifest
│   ├── home/luffy.png, home/strawhat.png
│   ├── images/                # mirror of assets/images for direct <img src="/images/..."> use
│   └── social/default-social.png
│
├── layouts/                   # ~160 .html files: Arbaaz overrides + the rest of Hextra
│   ├── home.html              # OVERRIDE — injects author block above content
│   ├── hextra-home.html       # OVERRIDE — invoked when `layout: hextra-home` is set
│   ├── single.html            # OVERRIDE — injects author block
│   ├── list.html              # OVERRIDE — injects author block
│   ├── blog/single.html       # OVERRIDE — author block + date + reading time
│   ├── docs/single.html       # OVERRIDE — author block + breadcrumb
│   ├── _partials/
│   │   ├── author-block.html  # OVERRIDE — Blowfish-style author render
│   │   ├── icon.html          # OVERRIDE — Blowfish icon partial
│   │   ├── badge.html         # OVERRIDE — Blowfish badge partial
│   │   ├── footer.html        # OVERRIDE — Hextra footer + custom slot retained
│   │   ├── functions/uid.html # OVERRIDE — deterministic UID (used by accordion/carousel)
│   │   ├── article-link/, article-meta/, hugo-embedded/   # Blowfish helper partials
│   │   ├── custom/            # empty slot — Hugo convention for site-level injection
│   │   └── (everything else is from Hextra: sidebar, navbar, head, search, opengraph, …)
│   └── _shortcodes/
│       └── (34 OVERRIDE .html files — see "Shortcodes" below — Hextra's shortcodes for everything else)
│
└── exampleSite/               # demonstrative downstream site (Hugo never reads this dir from a loaded theme)
    ├── hugo.toml              # full working site config — copy this into your own site
    ├── content/
    │   ├── _index.md          # homepage with `layout: hextra-home`
    │   ├── shortcodes-test.md # exercises every imported shortcode
    │   └── certifications/_index.md
    └── themes/
        └── hextras/     # Windows directory junction → ../../..  (so the example builds in-place)
```

---

## Shortcodes

All 34 are loaded automatically; use them in any markdown file with `{{</* name args */>}}` syntax. Where Hextra ships a shortcode of the same name, the hextras override at `layouts/_shortcodes/<name>.html` was kept during the merge, so this theme's version is the one Hugo finds.

| Shortcode        | What it renders                                                                                  |
| ---------------- | ------------------------------------------------------------------------------------------------ |
| `accordion`      | Collapsible group container (works with `accordionItem`).                                        |
| `accordionItem`  | One collapsible row — `title="…"`, `icon="…"`, `open=true`.                                      |
| `alert`          | Boxed callout. Positional arg = icon name (e.g. `circle-info`, `triangle-exclamation`).          |
| `article`        | Renders an article card link (used by section list pages).                                       |
| `badge`          | Small rounded label.                                                                             |
| `button`         | Styled `<a>` button — `href`, `target`, `download`.                                              |
| `carousel`       | Sliding image carousel.                                                                          |
| `chart`          | Chart.js block — pass JSON config inside the shortcode body.                                     |
| `codeberg`       | Embeds a Codeberg repo card (`user/repo`).                                                       |
| `codeimporter`   | Imports a remote code snippet — `url`, `from`, `to`, `type` (lang).                              |
| `external`       | Wraps content in an external-link badge.                                                         |
| `figure`         | Replacement for Hugo's built-in `figure`; supports caption, alt, link.                           |
| `forgejo`        | Forgejo repo card.                                                                               |
| `gallery`        | Grid of images — wraps `<img>` children, lazy-loaded.                                            |
| `gist`           | GitHub gist embed — `user/id`.                                                                   |
| `gitea`          | Gitea repo card.                                                                                 |
| `github`         | GitHub repo card — `user/repo`.                                                                  |
| `gitlab`         | GitLab repo card.                                                                                |
| `icon`           | Inline SVG icon by name — `{{</* icon "github" */>}}`.                                           |
| `katex`          | Inline KaTeX block.                                                                              |
| `keyword`        | Single keyword chip (used inside `keywordList`).                                                 |
| `keywordList`    | Comma-separated list of keyword chips.                                                           |
| `lead`           | Larger, lighter intro paragraph.                                                                 |
| `list`           | Compact bulleted list with custom bullet glyph.                                                  |
| `ltr` / `rtl`    | Force text direction on a block.                                                                 |
| `mdimporter`     | Imports a remote Markdown file inline.                                                           |
| `mermaid`        | Mermaid diagram block.                                                                           |
| `screenshot`     | Stylised screenshot frame around an image.                                                       |
| `swatches`       | Colour-swatch palette display.                                                                   |
| `timeline`       | Timeline container (works with `timelineItem`).                                                  |
| `timelineItem`   | One timeline entry — `icon`, `header`, `subheader`.                                              |
| `typeit`         | TypeIt.js typewriter animation — pass text in the body.                                          |
| `youtubeLite`    | Lazy-loaded YouTube embed — `id="…"`, `label="…"`.                                              |

A live exercise of every shortcode lives in [exampleSite/content/shortcodes-test.md](exampleSite/content/shortcodes-test.md).

### Adding a new shortcode

Drop a new file at `layouts/_shortcodes/<name>.html` and reference it as `{{</* <name> */>}}`. To override a Hextra shortcode, name your file the same as the upstream one — anything you place in the *site*'s `layouts/_shortcodes/` wins over the theme's copy.

---

## Auto-card list (`layout: cards`)

Tired of maintaining a giant `{{</* cards */>}}` block by hand and editing it every time you publish? Set one front-matter line on a section's `_index.md` and the page becomes a self-updating card grid of every child page.

```yaml
# content/writeups/_index.md
---
title: "Writeups"
layout: cards
cols: 3                                  # optional, default 3
orderBy: weight                          # weight | date | date-rev | title (default: weight)
defaultCardImage: "/images/writeups/featured.jpg"
cardImageStyle: "object-fit:cover; aspect-ratio:16/9;"
description: "Auto-listed from sub-pages."
---

Optional intro markdown rendered above the grid.
```

Each child page (a regular `.md` or a sub-section's `_index.md`) shows up as one card. Per-child front matter:

```yaml
---
title: "HackTheBox: AirTouch"
description: "Wireless attack chain — WPA-PEAP capture to internal pivot."
weight: 1                                # ordering within the parent
cardImage: "/images/writeups/airtouch.jpg"   # optional, falls back to defaultCardImage
cardSubtitle: "Wireless attack chain"        # optional, falls back to description
cardIcon: "shield"                       # optional Hextra icon name
cardTag: "new"                           # optional badge
cardTagColor: "blue"
cardImageStyle: "..."                    # optional per-card override
hidden: true                             # exclude from the grid (omit otherwise)
---
```

**Nested lists** work transparently: a sub-section's `_index.md` can itself set `layout: cards` and the user clicks down through the tree. Mix and match — a sub-section that needs a custom write-up can omit the line and gets the default Hextra section view instead.

The image is resolved in this order: page resource (e.g. an image sitting next to the markdown file) → site asset/static path → absolute URL. So both `cardImage: "cover.jpg"` (sibling file) and `cardImage: "/images/writeups/foo.jpg"` (absolute static path) work.

A live example lives in the example site at `exampleSite/content/writeups/` — `/writeups/` auto-lists `HTB` and `Wiz` as cards; `/writeups/HTB/` auto-lists `airtouch` and `browsed`; `/writeups/Wiz/` falls back to the default list view.

> **How it works under the hood:** [layouts/_default/cards.html](layouts/_default/cards.html) is a section/list layout selected by `layout: cards`. It walks `.Pages` (direct children only — not recursive), drops anything with `hidden: true`, sorts by your `orderBy` choice, then calls Hextra's existing `partials/shortcodes/card` partial for each — same look as a hand-written `{{</* card */>}}`, no shortcode round-trip required.

---

## The author block

A single `[params.author]` block in your site's `hugo.toml` drives the author render on every page:

```toml
[params.author]
  name     = "Your Name"
  email    = "you@example.com"
  image    = "images/profile.png"   # resolved via assets pipeline
  headline = "Your one-line title"
  bio      = "Optional longer paragraph."
  links = [
    { mail     = "mailto:you@example.com" },
    { github   = "https://github.com/you" },
    { linkedin = "https://linkedin.com/in/you" },
    # any iconName: url pair where iconName matches data/icons.yaml or assets/icons/*.svg
  ]
```

Toggles (all default to sensible values):

```toml
[params.article]
  showAuthor       = true     # render above content on single / blog / docs / list / home pages
  showAuthorBottom = false    # also render below content
[params.homepage]
  showAuthor       = true     # render on the home / hextra-home layout
```

Per-page override of just the headline via front matter:

```yaml
---
title: My post
authorHeadline: "Special headline for this post"
---
```

The block is implemented in [layouts/_partials/author-block.html](layouts/_partials/author-block.html) and injected by:

- [layouts/home.html](layouts/home.html)
- [layouts/hextra-home.html](layouts/hextra-home.html)
- [layouts/single.html](layouts/single.html)
- [layouts/list.html](layouts/list.html)
- [layouts/blog/single.html](layouts/blog/single.html)
- [layouts/docs/single.html](layouts/docs/single.html)

To **disable globally**, set `[params.article] showAuthor = false` and `[params.homepage] showAuthor = false`. To **change the look**, edit `author-block.html`.

---

## Adding custom code in your own site

Per Hugo's theme inheritance, anything you put in your own site's directories overrides the same path in this theme. Common extension points:

| You want to…                     | Put a file at…                                                       |
| -------------------------------- | -------------------------------------------------------------------- |
| Add site-wide CSS                | `assets/css/custom.css` (this theme already provides one — overriding it replaces it) |
| Add a new shortcode              | `layouts/_shortcodes/<name>.html`                                    |
| Override a layout (e.g. footer)  | `layouts/_partials/footer.html` or `layouts/_partials/custom/footer.html` |
| Inject HTML before `</head>`     | `layouts/_partials/custom/head-end.html`                             |
| Replace the author block         | `layouts/_partials/author-block.html`                                |
| Add a new icon                   | `assets/icons/<name>.svg` (any `{{</* icon "<name>" */>}}` reference resolves it) |
| Replace favicons                 | `static/favicon.ico` (and friends)                                   |
| Add a brand-new layout           | `layouts/<section>/<kind>.html`                                      |

The Hextra docs cover every other extension point: <https://imfing.github.io/hextra/docs/>.

---

## Modifying the theme itself

If you fork hextras and want to evolve it:

1. **Add or override a layout** — drop the file at the matching path under `layouts/`. Since Hextra is bundled inside this repo (not a separate theme), Hextra files live alongside the overrides; just edit them in place or replace them.
2. **Add a shortcode** — `layouts/_shortcodes/<name>.html`. Hugo resolves `{{</* name */>}}` to it automatically.
3. **Add an icon** — drop the SVG in `assets/icons/<name>.svg`, then add a corresponding entry in `data/icons.yaml`:

   ```yaml
   <name>: <single-line svg markup>
   ```

   The icons file already merges Hextra's full set with the Blowfish additions (339 entries, no overlap). New entries go in the "Blowfish-origin additions" block at the bottom.
4. **Bundle a new asset** — `assets/images/<file>` (processable, gets a content hash) or `static/<file>` (served as-is).
5. **Pull upstream Hextra updates** — Hextra is vendored, not pinned. To refresh:
   ```bash
   git clone --depth 1 https://github.com/imfing/hextra.git /tmp/hextra
   cp -rn /tmp/hextra/{layouts,assets,data,static,i18n} ./   # -n = never overwrite our overrides
   # then merge any new icons from /tmp/hextra/data/icons.yaml into ours
   ```
6. **Test before pushing**:

   ```bash
   cd exampleSite && hugo --minify
   # then open public/index.html and public/shortcodes-test/index.html
   ```

The `exampleSite/` is wired with a Windows directory junction (`exampleSite/themes/hextras` → repo root), so edits to the theme are picked up by the next `hugo server` reload without copying anything.

> On Linux/macOS, replace the junction with a symlink:
> ```bash
> ln -s ../../.. exampleSite/themes/hextras
> ```

---

## Files in this repo, at a glance

| Path                             | Purpose                                                                   |
| -------------------------------- | ------------------------------------------------------------------------- |
| `theme.toml`                     | Theme manifest read by Hugo / Hugo Showcase.                              |
| `hugo.toml`                      | Theme-level config — markup, output formats, Hugo-version pin.            |
| `go.mod`                         | Module path for Hugo Modules consumers (Path B above).                    |
| `LICENSE`                        | MIT (this theme), with upstream Hextra and Blowfish attribution.          |
| `CHANGELOG.md`                   | Human-readable record of changes.                                         |
| `archetypes/default.md`          | Template used by `hugo new`.                                              |
| `assets/`, `static/`, `data/`    | Personal assets and merged icon data.                                     |
| `layouts/`                       | Layout overrides + Blowfish-ported partials and shortcodes.               |
| `exampleSite/`                   | A working downstream site demoing every theme feature.                    |
| `_tools/`                        | Build-time helpers (icon merger, Tailwind prefix script). Gitignored.     |

---

## Upstream attribution

- **Hextra** — <https://github.com/imfing/hextra> (MIT) — base theme.
- **Blowfish** — <https://github.com/nunocoracao/blowfish> (MIT) — origin of the ported shortcodes, icon set, and author-block design.

This theme would not exist without either project.

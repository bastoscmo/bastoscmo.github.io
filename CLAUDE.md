# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

This is the source for **bastoscmo.github.io**, a personal academic homepage built on the [Academic Pages](https://academicpages.github.io/) Jekyll theme (a fork of Minimal Mistakes, theme variant `air`). It is a static Jekyll site deployed via GitHub Pages — there is no application backend or test suite; "correctness" means the site builds cleanly and renders as expected.

## Running locally

Ruby/Jekyll and Node are both required (Node is only for the JS bundling step).

```bash
bundle install                             # install Ruby gems (delete Gemfile.lock and retry if it fails)
bundle exec jekyll serve -l -H localhost   # serve at localhost:4000, live-reloads on .md/.html changes
```

Changes to `_config.yml` require stopping and restarting Jekyll — it is not hot-reloaded.

### Docker alternative

```bash
docker compose up
```

Builds from `Dockerfile` and serves at `localhost:4000` using `_config.yml` + `_config_docker.yml` (bind-mounts the whole repo, runs as uid 1000). A `.devcontainer/` config exists for VS Code Dev Containers using the same compose file.

### JS build

`assets/js/main.min.js` (loaded as an ES module by `_includes/scripts.html`) is a concatenation of jQuery, fitvids, jquery-smooth-scroll, `assets/js/plugins/jquery.greedy-navigation.js`, and `assets/js/_main.js`, produced by uglify-js:

```bash
npm run build:js     # one-off build via uglifyjs
npm run watch:js      # rebuild on change to assets/js/**/*.js
```

Edit `assets/js/_main.js`, not `main.min.js` directly — it's generated. Note `assets/js/theme.js` (Plotly light/dark templates) is *not* in the uglify bundle and is `import`ed at runtime by `_main.js` — this only works because `main.min.js` is loaded with `type="module"`, so keep it a separate file rather than folding it into the bundle. `assets/js/collapse.js` and `assets/css/collapse.css` exist but aren't referenced by any include or the build script — dead code, not currently wired to anything.

## Content architecture

Content lives in Jekyll collections, each a directory of Markdown files with YAML front matter, split into per-year subdirectories (e.g. `_publications/2025/`, `_talks/2025/`, `_teaching/2025/`, `_posts/2026/`). **This year-subdirectory layout is a convention specific to this fork, not stock Academic Pages**, and it is not fully honored by the tooling — see "Known gotchas" below.

- `_publications/` — journal articles / conference papers, rendered via `_pages/publications.md`. `category:` front matter (`manuscripts`, `conferences`, `books`) must match a key under `publication_category` in `_config.yml`, which controls the heading grouping on the page.
- `_talks/` — talks/seminars, rendered with `_layouts/talk.html`.
- `_teaching/` — courses taught.
- `_posts/` — blog posts, standard Jekyll date-prefixed filenames, listed at `/year-archive/`.
- `_portfolio/` — portfolio/project entries (this one is *not* split by year — still flat).
- `_pages/` — standalone pages (`cv.md`, `about.md`, `publications.md`, `talks.md`, `teaching.md`, `development.md` + its per-project subpages, taxonomy archive pages, etc.). Pages are only picked up because `_config.yml`'s `include:` explicitly lists `_pages`.

Every content file needs a `permalink:` plus whatever front matter that collection's layout expects (e.g. `venue`, `citation`, `excerpt` for publications; `location`, `venue`, `talk_type` for talks — `location` specifically is what the talk-map geocoder reads). Copy an existing file in the same collection/year rather than starting from scratch. New files go in the current year's subdirectory (create it if it doesn't exist yet).

### `/development/` project pages

`_pages/development.md` is a hub page linking to `_pages/siesta-toolbox.md`, `_pages/wantibexos.md`, `_pages/aims-toolbox.md`. These document external software projects (SIESTA/Wantibexos/FHI-aims tooling) the author maintains elsewhere — they are documentation pages, not the actual project source, which does not live in this repo.

### CV

`/cv/` (`_pages/cv.md`, `layout: archive`) is the only CV page. It's hand-written Markdown; its Publications/Talks/Teaching sections are populated live at build time straight from the Jekyll collections (`{% for post in site.publications reversed %}` etc., via `_includes/archive-single-cv.html` / `archive-single-talk-cv.html`) — always accurate, no generation step, no separate data file to keep in sync.

There used to be a second, JSON-Resume-based `/cv-json/` page (rendering a generated `_data/cv.json` via a Python script). It was removed: the generator was never actually run for this site — the checked-in `_data/cv.json` was 100% the stock Academic Pages demo data ("Your Sidebar Name", "Paper Title Number 1", etc.), not Carlos's real CV — and its "Download CV as PDF" button pointed at a `/files/cv.pdf` that doesn't exist. If a JSON/PDF export is wanted again in the future, rebuild the generator rather than reintroducing the old one — it also had a non-recursive `glob.glob(os.path.join(dir, "*.md"))` that would find zero files in `_publications/`/`_talks/`/`_teaching/` now that content lives in year subdirectories.

### Talk map

`talkmap.py` / the equivalent cell in `talkmap.ipynb` now glob recursively (`glob.glob("_talks/**/*.md", recursive=True)`) so they find talks in the per-year subdirectories. This was previously non-recursive and found zero files; the checked-in `talkmap/org-locations.js` had never actually been generated from real data at all — it was still the stock Academic Pages demo locations ("UC-Berkeley Institute for Testing Science" etc.), same situation as the old `/cv-json/` page. It's now been regenerated for real from the 3 talks in `_talks/`. The GitHub Actions workflow (`.github/workflows/*.yml`, triggers on push to `_talks/**`/`talkmap.ipynb`) re-executes the notebook and commits `talkmap_out.ipynb` on every relevant push, so keep it that way going forward.

`/talkmap.html` (embeds `talkmap/map.html` in an iframe; not linked from nav — `talkmap_link: false` in `_config.yml`, but reachable directly) previously rendered no markers at all, even with correct data: `talkmap/leaflet_dist/leaflet.markercluster-src.js` was vendored from 2012–2013 (pre-Leaflet-1.0) while `map.html` loaded core Leaflet from cdnjs pinned to `1.0.0-beta.2` — incompatible, so `MarkerClusterGroup`'s internal `_featureGroup` (what actually gets drawn) stayed empty even though marker data loaded fine into `addressPoints` and the cluster tree (confirmed via browser JS console, not just reading the code).

Fixed by bumping `talkmap/map.html` to Leaflet **1.9.4** (cdnjs) and re-vendoring `talkmap/leaflet_dist/` with modern Leaflet.markercluster **1.5.3** (also switched the Esri tile URL to `https://`, since the old `http://` one gets blocked/upgraded on this HTTPS-served site). **Important**: `getorg.orgmap.output_html_cluster_map()` — the function `talkmap.py`/`talkmap.ipynb` used to call — rewrites `map.html` and everything under `leaflet_dist/` from templates hardcoded inside the installed `getorg` package itself (still the same 2012-2013 code), so calling it again would silently undo this fix. Both scripts now call `getorg.orgmap.location_dict_to_jsvar(...)` directly instead, which only (re)writes `org-locations.js` — `map.html` and `leaflet_dist/` are hand-maintained from here on and won't be touched by future runs.

### Publication/talk TSV import

`markdown_generator/` holds notebooks/scripts (`publications.ipynb`/`.py` + `publications.tsv`, `talks.ipynb`/`.py` + `talks.tsv`, `OrcidToBib.ipynb`, `PubsFromBib.ipynb`/`pubsFromBib.py`) for bulk-generating front-matter Markdown files from tabular/BibTeX sources. These scripts write output flat into `../_publications/` / `../_talks/` (no year subdirectory logic) — after running one, manually move the generated files into the correct year folder to match the convention the rest of the repo uses. For one-off additions, skip this tooling and just copy an existing Markdown file's front matter instead.

## Site-wide configuration

- `_config.yml` — author/social-profile sidebar data (`author:` block), `site_theme` variant (`"air"` vs `"default"`), `publication_category` headings, comments provider, analytics, Jekyll `collections:`/`defaults:` (which layout + which of `author_profile`/`share`/`comments`/`related` each collection gets by default). **Not** hot-reloaded by `jekyll serve` — restart after editing.
- `_data/navigation.yml` — the top nav (masthead) links and their order. This is where to add/remove/reorder nav items, **not** `_config.yml`.
- `_data/ui-text.yml` — locale-keyed UI strings (labels like "Read more", meta labels) used throughout `_includes/`.
- `_data/authors.yml` — optional per-post/page author overrides (separate from the single site-wide `author:` block in `_config.yml`); currently empty scaffolding, unused since every page is by the one site author.
- `_data/comments/` — Staticman-generated comment YAML files, one subfolder per commented post/page slug; not something to hand-author.

### SEO / social preview

`site.description`/`site.og_description` (in `_config.yml`) are the fallback `<meta name="description">`/`og:description` used by `_includes/seo.html` whenever a page has no `excerpt:` of its own — page excerpts always take priority when present. `site.og_image` (`images/og-image.png`, a generated 1200×630 brand card) is the fallback `og:image` for any page without its own `header.image`/`header.overlay_image`; `_includes/seo.html` needed a small patch to add that fallback, since upstream Minimal Mistakes only used `site.og_image` for `twitter:image` and a JSON-LD logo, never for the actual `og:image` tag — don't assume setting `og_image` in config alone is enough after a template update/merge from upstream.

## Runtime page features (site-wide, via `_includes/footer/custom.html`)

Every page loads MathJax, Plotly, and Mermaid unconditionally:

- **Math**: plain LaTeX (`$...$`, `$$...$$`) in any Markdown content is rendered by MathJax — no per-page opt-in needed.
- **Diagrams**: a fenced code block with the `mermaid` language (` ```mermaid `) is rendered in place by Mermaid on page load.
- **Charts**: a fenced code block with the `plotly` language (` ```plotly `) containing Plotly JSON (`{"data": [...], "layout": {...}}`) is picked up by `assets/js/_main.js`, hidden, and re-rendered as a live Plotly chart — the layout `template` is swapped automatically between `plotlyLightLayout`/`plotlyDarkLayout` (`assets/js/theme.js`) to match the current site theme.

Also from `_includes/head/custom.html`: Academicons (`assets/css/academicons.css`) are available for academic-profile icons (ORCID, Google Scholar, etc.) beyond the stock Font Awesome set.

## Dark/light theme

`assets/js/_main.js` implements a manual theme toggle (`#theme-toggle` button, `#theme-icon`) independent of Jekyll: it stores `"dark"`/`"light"`/`"system"` in `localStorage`, sets `data-theme="dark"` on `<html>` (absent = light), and falls back to `prefers-color-scheme` when unset. SCSS has matching light/dark partials per theme variant: `_sass/theme/_air_light.scss` / `_air_dark.scss` (and `_default_light.scss` / `_default_dark.scss` for the unused `default` variant) — if you touch theme colors, update both the light and dark partial for the active variant (`air`), and remember Plotly chart theming (above) is driven by the same computed theme.

## Styling

SCSS lives in `_sass/` (`theme/`, `layout/`, `include/`, `vendor/` for Breakpoint/Susy/Font Awesome) and is compiled by Jekyll's built-in Sass support from `assets/css/main.scss` (`.sass-cache/` is a build cache, not source — safe to delete). Page-level layouts are in `_layouts/`; shared partials (head, footer, comment providers, analytics providers) are in `_includes/`, with `_includes/head/custom.html` and `_includes/footer/custom.html` as the intended site-specific customization points (as opposed to editing the stock `head.html`/`footer.html`).

## Build/deploy notes

- `_site/` is the generated Jekyll output — never edit it directly, it's overwritten on every build.
- Files meant for direct download links (PDFs, zips, slides) go under `files/`, organized by year/type (`files/2025/`, `files/publication/`, `files/talks/`, `files/teaching/`), and are served at `/files/...` because `_config.yml`'s `include:` also lists `files`.
- GitHub Pages builds via the standard `pages-build-deployment` workflow (see badge in README) — the only custom workflow is the talkmap scraper described above.

# Phase 1: Audit

Source: `scripts/audit-hugo.sh` output plus manual review. SSG detected: **hugo** (`npx @cloudcannon/cli configure detect-ssg`).

## Checklist

- [x] Hugo version, modules, themes
- [x] Bookshop detected → read `cloudcannon-visual-editing/migrating-from-bookshop.md` + `hugo/migrating-from-bookshop.md`
- [x] npm tooling
- [x] Content sections, front matter fields, routing overrides, taxonomies
- [x] Data files and which templates read them
- [x] Pages and routing
- [x] Layouts and partials, shortcodes, asset pipeline, `.Content` in partials
- [x] Classifying static pages + census table
- [x] Build pipeline
- [x] Flags and special patterns
- [x] Sectioning recommendation

## 1. Hugo version, modules and themes

| Item | Finding |
| --- | --- |
| Hugo pinned on CloudCannon | `.cloudcannon/initial-site-settings.json` has `"hugoVersion": "0.128.1"` — **misspelled key, ignored** by CloudCannon (schema key is `build.hugo_version`). |
| Local Hugo used for migration | 0.166.0 extended (the user supplied it; the upgrade decision was made up front). |
| Upgrade result | The unmodified site **fails** on 0.166: `resources.ToCSS` removed. Fixed with `css.Sass` in `layouts/_default/baseof.html`. Also fixed deprecations: `.Site.Data` → `hugo.Data` (navbar, footer) and `languageCode` → `locale` (config.toml). The remaining `.Site.Data` warning came from the Bookshop module. After the fixes the unmodified site builds 25 HTML pages; that build is the **parity baseline**. |
| Module project | Yes — `go.mod` (`module sendit.local`, go 1.17). |
| Templates come from | Project `layouts/` + the Bookshop local module `component-library/` (mounted to `layouts/partials/bookshop`) + `github.com/cloudcannon/bookshop/hugo/v3` (module cache). No theme. |
| Bookshop | **Yes.** 16 `*.bookshop.yml` components, `@bookshop/*` npm deps, `npx @bookshop/generate` in `.cloudcannon/postbuild`, `_bookshop_name` in 5 content files, `bookshop_bindings`/`bookshop_partial` in `_default/list.html` and `_default/single.html`. No Bookshop entries in `.gitignore`. No co-located component SCSS (all styles already live in `assets/scss/components/`). |
| npm tooling | `package.json` has only Bookshop deps and `npm-run-all`. `build` = `hugo --source ./ --destination public --baseURL /`. No Tailwind/PostCSS/Pagefind. |

## 2. Content sections

| Section | Files | Layout | `_index.md` | Notes |
| --- | --- | --- | --- | --- |
| (root) | `_index.md`, `about.md`, `contact.md`, `feature.md`, `pricing.md` | `_default/list.html` (home), `_default/single.html` (others) — both Bookshop page builders over `content_blocks` | `content/_index.md` is the home page | Page builder pages |
| `blog` | 6 posts + `_index.md` | `blog/single.html`, `blog/list.html` → `partials/blog-list.html` | Yes; drives list title + description | `_index.md` has `url: /blog/` (redundant with default) |
| `privacy` | `_index.md` only | `privacy/list.html` | Yes — the whole page | Section of one; structured `privacy` array (descriptive / bulletpoints) + `hero.title` |

No leaf bundles. Front matter is YAML everywhere (13 files).

### Front matter fields

**Blog posts** (`content/blog/*.md`, all 6):

| Key | Type | Always? | Read by |
| --- | --- | --- | --- |
| `title` | string | yes | `blog/single.html`, `blog-list.html`, meta |
| `date` | datetime | yes | `blog/single.html` (`.Date`) |
| `author` | string | yes | `blog/single.html` |
| `categories` | string[] (curated in `data/blog-tags.yaml`) | yes | single + list cards (links to `/categories/<term>/`) |
| `thumbImg.image_path` | image | yes | list cards, recent posts |
| `featuredImg.image_path` | image | yes | `blog/single.html` |
| `seo.{page_description,canonical_url,featured_image,author_twitter_handle,open_graph_type,no_index}` | object | yes | `partials/meta.html` |
| `draft` | bool | yes | Hugo |
| `_schema` | string | 1 file (`default`) | CloudCannon |
| body | markdown | yes | `.Content` |

**Page builder pages**: `title`, `seo.*`, `content_blocks[]` (each with `_bookshop_name`).
**Blog `_index.md`**: `title`, `description`, `url`, `seo.*`.
**Privacy `_index.md`**: `title`, `seo.*`, `hero.title`, `privacy[]` of `{key: descriptive|bulletpoints, heading, texts[]: {value}}`.

- Routing overrides: `url: /blog/` in `content/blog/_index.md`. No `permalinks`, no `slug:`.
- Taxonomies: Hugo defaults (`categories`, `tags`). `categories` values are curated in `data/blog-tags.yaml` (select source). `tags` unused (generated empty `/tags/`).

### Data files

| File | Shape | Read by |
| --- | --- | --- |
| `data/nav.yaml` | object: `logo`, `items[]` (`link`, `text`, `enable_dropdown`, `dropdown[]`), `enable_nav_btn`, `nav_btn` | `partials/navbar.html` |
| `data/footer.yaml` | object: `logo`, `logo_url`, `copyright`, `social[]`, `sections[]` (`title`, `links[]`) | `partials/footer.html` |
| `data/meta.yaml` | object: SEO defaults | `partials/meta.html` (head only — not visible) |
| `data/blog-tags.yaml` | top-level string array | CloudCannon select source for `categories` only |

All like-shaped lists are arrays (no keyed-map anti-pattern).

## 3. Pages and routing

| URL | Source | Kind |
| --- | --- | --- |
| `/` | `content/_index.md` | Home, page builder (`_default/list.html`) |
| `/about/` | `content/about.md` | Page builder |
| `/contact/` | `content/contact.md` | Page builder |
| `/feature/` | `content/feature.md` | Page builder |
| `/pricing/` | `content/pricing.md` | Page builder |
| `/privacy/` | `content/privacy/_index.md` | Section list page, hardcoded layout reading structured front matter |
| `/blog/` | `content/blog/_index.md` | List page, paginated 9 (`.Paginator 9`), `/blog/page/1/` alias |
| `/blog/<slug>/` ×6 | `content/blog/*.md` | Blog post |
| `/categories/`, `/categories/<term>/` ×3 | generated | Taxonomy — leave as layouts |
| `/tags/` | generated | Taxonomy (empty) — leave |
| `/404.html` | `layouts/404.html` | Layout-only — leave |

Pagination aliases: `page/1/` redirect pages (6 aliases). No `aliases` in front matter.

## 4. Layouts and partials

- **Base layout** `_default/baseof.html`: `meta.html` partial, vendor CSS/JS from `static/vendor`, SCSS via asset pipeline (`resources.Get` → `css.Sass` → Minify → Fingerprint), `navbar.html`, `main` block, `footer.html`.
- **Page templates**
  - `_default/list.html`, `_default/single.html`: Bookshop dispatch (`bookshop_bindings` + `bookshop_partial "page"`). Become the page-builder `range`.
  - `blog/single.html`: **inline markup** — title, date, author, reading time, word count, featured image, `.Content`, then a "Recent Blog" grid (first 3 posts; heading hardcoded).
  - `blog/list.html` → `partials/blog-list.html` (hero title/description + paginated cards).
  - `privacy/list.html`: **inline markup** — hero title + `privacy[]` cards + a scroll-spy nav built from the same array.
  - `_default/taxonomy.html` → `blog-list`.
  - `404.html`: hardcoded.
- **Partials**: `navbar.html` (`hugo.Data.nav`), `footer.html` (`hugo.Data.footer`), `meta.html` (head), `blog-list.html` (list pages).
- **Bookshop components** (16, `component-library/components/<group>/<name>/<name>.hugo.html`): home/hero, home/video, global/counter, global/header, global/feature, global/testimonial, global/faq, about/hero, about/video, about/team, contact/hero, contact/form, feature/hero, pricing/hero, pricing/table, privacy/hero (privacy/hero is not used by any content file).
- **Shortcodes**: none defined, none used in content. **No snippets needed.**
- **Render hooks**: none.
- **Asset pipeline**: one call, `resources.Get "/scss/theme.scss"` in `baseof.html` (a page template, not a partial — no `ENV_CLIENT` guard needed). All images are in `static/images/` (uploads path `static/images`).
- **`.Content` in partials**: none. `.Content` only in `blog/single.html`.

Visual-editing candidates: every page-builder block (heroes, counters, features, testimonials, FAQ, pricing, team, video, contact form), the blog post title/author/image/body, blog list hero, privacy hero + items, navbar and footer text/links (data files). Sidebar-only: SEO fields, `meta.yaml`, `reversed`/`alternateStyle`/`no_top_pad` toggles, `placer`, nav `enable_dropdown`.

### Classifying static pages — census

| Page file | Distinct content sections | Layout repeated on other pages? | Editor will add similar pages? | Recommended pattern | Needs editable region? |
| --- | --- | --- | --- | --- | --- |
| `content/_index.md` (home) | hero, counter, video, header, 3× feature, testimonial | Blocks shared with feature/about | Yes | Page builder (`pages` collection) — already Bookshop | Yes — every block |
| `content/about.md` | hero, counter, video, team | Blocks shared | Yes | Page builder | Yes |
| `content/contact.md` | hero, contact form | — | Maybe | Page builder | Yes |
| `content/feature.md` | hero, header, 3× feature, testimonial | Blocks shared with home | Yes | Page builder | Yes |
| `content/pricing.md` | hero, pricing table, FAQ | — | Maybe | Page builder | Yes |
| `content/privacy/_index.md` + `layouts/privacy/list.html` | hero title, structured legal sections, scroll-spy nav | Terms-type pages | Unlikely | Legal page in `pages` collection with its own fixed schema (keeps the structured `privacy` array — it drives the scroll-spy nav) | Yes — hero title, headings, texts |
| `content/blog/_index.md` | hero title + description, generated list | — | No | Section `_index.md` with its own schema | Yes — title, description |
| `layouts/blog/single.html` (6 posts) | title, meta, featured image, body, recent posts | All posts | Yes | Fixed-schema `blog` collection | Yes — title, author, featured image, body |
| `layouts/404.html` | image, one line, button | — | No | Leave as layout | No — no file to write back to |
| taxonomy pages | generated | — | No | Leave | No |
| `data/nav.yaml`, `data/footer.yaml` | header nav, footer | Every page | — | Shared UI → data files | Yes — nav/footer text and link arrays |
| "Recent Blog" heading in `blog/single.html` | one hardcoded string | Every post | — | Leave hardcoded (see visual-editing.md) | Justify |

## 5. Build pipeline

- `npm run build` → `hugo --source ./ --destination public --baseURL /`.
- `.cloudcannon/postbuild`: `rm -rf node_modules; npm i; npx @bookshop/generate` — Bookshop-only; the hook goes away entirely.
- No `config/<env>/` directories, no `hugo.Environment` checks.
- CLI suggests: install `npm i`, build `npm run build`, output `public`, preserved `node_modules/`, `resources/`, `.hugo_cache/`, env `HUGO_CACHEDIR`.

## 6. Flags and special patterns

- **Positional CSS selectors on future array containers** (a `<template>` blueprint shifts them):
  - `_counter-up.scss:64-65` — `.counter-up-content:nth-child(2|4)` inside `.counter-up-wrapper` (the `numbers` array).
  - `_root.scss:546/549` — `.accordion-flush .accordion-item:first-child/:last-child`. FAQ uses `.accordion` not `.accordion-flush` → not affected.
  - Others (`_blog`, `_navbar`, `_footer`, `_pricing`, `_services`, `_feature`) to be checked per region in Phase 4.
- **Global JS bindings** (`static/js/script.js`): `$(document).ready` — magnific-popup on `.popup-vimeo` (home/video, about/video), countUp via IntersectionObserver on `.counter` (global/counter), accordion "shows" class on `#accordionExample .accordion-item` (global/faq), nav toggles. Re-rendered blocks lose these in the editor (acceptable; editor-only).
- **Inline scripts/styles in partials**: none. `global/faq` has `onclick="accordionBorder()"` — the function doesn't exist anywhere (pre-existing site bug; left as-is).
- **Numbers displayed as text**: `global/counter` `numbers[].number` (200, 2016, 40, 4322), `pricing/table` `price` / `discountPrice` (0, 199, 125…). Must be quoted strings + `type: text` if bound to text regions.
- **Raw HTML in content**: none. Goldmark `unsafe` not set (default false).
- **Scroll-reveal / entrance animations**: none (no AOS/WOW).
- **Existing CMS/deploy config**: an existing `cloudcannon.config.yaml` (Bookshop-era, `.yaml` extension), `schemas/page.html`, `schemas/post.md`. No netlify/forestry/workflows.
- **Existing config issues noticed**: `_structures.content_blocks` has `style: modal` and **no `values`** (Bookshop generated them at build time). The `articles` collection has path `content/blog`, so it includes `_index.md`. `.bookshop.yml` `_inputs.image` keys don't match any field (the fields are `image_path`).

## 7. Sectioning recommendation

| Signal | Value | Threshold | Tripped? |
| --- | --- | --- | --- |
| Total pages | 13 content pages (25 HTML incl. taxonomy/pagination) | > 30 | No |
| Hardcoded template → YAML conversions | 0 (page builders already exist in content; privacy already front-matter driven) | > 15 | No |
| Distinct collections | 3 (`pages`, `blog`, `data`) | > 5 | No |

0/3 tripped → single pass. No `plan.md` needed.

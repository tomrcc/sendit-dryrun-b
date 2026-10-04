# Phase 4: Visual editing

## Hugo version

Upgraded 0.128.1 (CloudCannon setting, misspelled key, so CloudCannon was really building on its default) → **0.166.0** locally and in `build.hugo_version`. Upgrade fixes on the unmodified site before any integration work: `resources.ToCSS` → `css.Sass` (baseof), `.Site.Data`/`site.Data` → `hugo.Data` (navbar, footer, meta), `languageCode` → `locale`. The editor's in-browser renderer (module v0.0.21) embeds Hugo **0.164.0** (found in `hugo_renderer.wasm`), so `hugo.Data` is available there too. The re-rendered partials (navbar, footer) take their data as `.` anyway.

## Section census

Treatments per `astro/visual-editing.md § Section census`, with the Hugo **Partial?** column. All page-builder blocks are partials under `layouts/partials/<group>/<name>.html` and are re-rendered through the dispatcher's `array-item` + `data-component` wrapper; their nested regions use paths relative to the block.

| Page | Section | Partial? | Treatment | Binding plan | Data completeness | Justification |
| --- | --- | --- | --- | --- | --- | --- |
| Home, About, Contact, Feature, Pricing | Page builder (`content_blocks`) | Dispatcher in `_default/list.html` + `_default/single.html` (page templates — primitives only) | array + component per item | `<div data-editable="array" data-prop="content_blocks" data-component-key="_name">`; per item `<div data-editable="array-item" data-id="{{ ._name }}" data-component="{{ ._name }}">` | All in front matter | — |
| Home | `home/hero` | Yes | text + image + button text | `title` (span), `description` (text, markdownify), `data-prop-src="image_path"`, `link.text` | Decorative shape images hardcoded (presentation) | — |
| Home | `global/counter` | Yes | text + array | `title`, `title_suffix`, `description` (text); `numbers` array on `.counter-up-wrapper`; items: `prefix`, `number`, `suffix`, `text` | Counter numbers quoted | `alternateStyle` is a sidebar switch (class binding → block re-render) |
| Home | `home/video` | Yes | image | `data-prop-src="image_path"` | `videoUrl` is a link → sidebar | — |
| Home, Feature | `global/header` | Yes | text | `title`, `title_suffix`, `description` | `no_top_pad` sidebar switch | — |
| Home, Feature | `global/feature` ×3 | Yes | text + image + button text | `title`, `description`, `data-prop-src="image_path"`, `btn.text` in **both** `reversed` branches | Effect images hardcoded (presentation) | `reversed` sidebar switch |
| Home, Feature | `global/testimonial` | Yes | text + array | `title`, `title_suffix`, `description`; `slider` array on `.row`; items: `data-prop-src="image_path"`, `author`, `designation`, `message` | — | — |
| About | `about/hero` | Yes | text + array + button text | `title`, `description`, `link.text`; `hero_images` array on `.about-hero-two-banner` → **not possible as-is** (static `.pattern` sibling) → wrap items only (see below); items: `data-prop-src="image_path"` | `placer` → sidebar select (class binding) | — |
| About | `about/video` | Yes | text + image + button text | `title`, `description`, `btn.text`, `data-prop-src="video.image_path"` | `video.link` → sidebar | — |
| About | `about/team` | Yes | text + array | `title`; `team_members` array on `.row`; items: `data-prop-src="image_path"`, `name`, `designation` | — | — |
| Contact | `contact/hero` | Yes | text | `title`, `description` | — | — |
| Contact | `contact/form` | Yes | text | `address.heading`, `address.address`, `phone.heading`, `phone.cell`, `email.heading`, `email.email`, `form.heading`, `form.*.heading` labels, `form.submitBtn.text` | Input `placeHolder`s are attributes → sidebar | Placeholders can't host a text region |
| Feature | `feature/hero` | Yes | text + image + button text | `title`, `description`, `btn.text`, `data-prop-src="image_path"` | — | — |
| Pricing | `pricing/hero` | Yes | text | `title`, `description` | — | — |
| Pricing | `pricing/table` | Yes | array (nested array) + text | `pricing_tier` array on `.row`; items: `tier`, `description`, `price`/`discountPrice` (in both `hasDiscount` branches), `btn.text`; `feature` array on the `<ul>`, items: `item` | Prices quoted | `isActive`/`hasDiscount` sidebar switches (class/branch → block re-render) |
| Pricing | `global/faq` | Yes | text + array | `title`, `title_suffix`, `description`; `faq` array on `#accordionExample`; items: `title` (span), `description` (text) | — | — |
| (unused) | `privacy/hero` | Yes | text | `title` | — | Not on any page today; wired so it works when added |
| Privacy | Hero title | No — inline in `_default/privacy.html` (page template) | text | `hero.title` | — | — |
| Privacy | Legal sections + scroll-spy nav | No → **extract** to `layouts/partials/legal/sections.html` | component + array + text | `<editable-component data-component="legal/sections" data-prop="privacy">`; `privacy` array on `.scrollspy-example`; items: `heading`, `texts` array (paragraph variant: new wrapper `<div>` holding the `.card-body` items; bullet variant: the `<ul>`), text items `value` | Nav links are derived from the same headings (re-rendered by the component) | — |
| Blog listing | Hero title + intro | Yes (`blog-list.html`, shared with taxonomy pages) | text | `title`, `description` (span) — only when the page has a file (not on taxonomy term pages) | — | — |
| Blog listing | Post cards + pagination | Yes | sidebar-only | — | Cards come from each post's own file | Generated listing of other files, paginated; editing a card's title here would edit a different file. Posts are edited on their own page |
| Blog post | Title, author, featured image | No (page template) | text + image | `title` (span), `author` (span), `data-prop-src="featuredImg.image_path"` | — | — |
| Blog post | Date, reading time, word count | No | sidebar-only | — | — | Date is a formatted datetime (sidebar datetime input); reading time and word count are computed |
| Blog post | Body | No | text | `data-editable="text" data-prop="@content"` on the wrapper of `{{ .Content }}` | — | — |
| Blog post | "Recent Blog" heading | No | text (data file) | `@data[blog-post].recent_heading` | Moved to `data/blog-post.yaml` in Phase 3 | — |
| Blog post | Recent posts grid | No | sidebar-only | — | Other files | Generated from the three latest posts |
| All pages | Header / navigation | Yes (`navbar.html`) | data-file + component + array | `<editable-component data-component="navbar" data-prop="@data[nav]">` at the call in `baseof.html`; partial takes the nav data as `.`; `items` array on `.navbar-nav`, items: `text`; nested `dropdown` array on `.dropdown-menu`, items: `dropdown_text`; `data-prop-src="logo"`; `nav_btn.text` | Active-link state computed from `page` | Links, `enable_dropdown`, `enable_nav_btn` are sidebar fields |
| All pages | Footer | Yes (`footer.html`) | data-file + component + array | `<editable-component data-component="footer" data-prop="@data[footer]">` at the call; partial takes the footer data as `.`; `data-prop-src="logo"`; `copyright`; `social` array on `.social-icon ul` (icon class → re-render; no text); `sections` array — columns are siblings of the logo column inside `.row` → **no array region**; bind by index `sections.N.title` and each column's `links` array | Year is computed (`now`) | Section columns can't be an array region without adding a layout wrapper; add/remove columns in the sidebar |
| 404 | Everything | No | none | — | — | Layout-only page, no file to write back to (audit decision) |
| Taxonomy term pages | Everything | shared `blog-list.html` | none | — | — | Generated, no file |

All census rows are implemented as written, with one change: the About hero's `hero_images` array got a plain static `<div data-editable="array">` wrapper around the images only, leaving the `.pattern` decoration outside. The images are absolutely positioned with descendant selectors against `.about-hero-two-banner`, so a static wrapper doesn't move them.

## Setup (infrastructure checklist)

- [x] Hugo 0.166.0 locally, `build.hugo_version: "0.166.0"`
- [x] `go.mod` requires `github.com/CloudCannon/editable-regions v0.0.21` (release tag). `hugo mod tidy` dropped Bookshop from `go.mod`/`go.sum`
- [x] `config.toml` imports only the editable-regions module (Bookshop imports and the `local/component-library` replacement removed). There were no `module.mounts` in the site config
- [x] `{{ partial "editable-regions" . }}` in `<head>` of `_default/baseof.html` (the only base layout)
- [x] `assets/jsconfig.json` gitignored (already)
- [x] Clean build: `public/_cloudcannon/` has `hugo_renderer.wasm.<hash>.gz`, `live-editing.<hash>.js` (and `hugo-worker.<hash>.js`)
- [x] Every component to re-render is a partial: 16 blocks, `navbar`, `footer`, `legal/sections` (new — extracted from the privacy page template)
- [x] Asset pipeline: the only `resources.Get` is in `baseof.html` (page template, never bundled); all images are plain `<img src>` from `static/`, so no `ENV_CLIENT` image guards or static→assets mount needed
- [x] No component partial renders `.Content`
- [x] Nothing vendored — all templates are in the project's `layouts/`

## Bookshop removal (migrating-from-bookshop.md checklist)

- [x] Structures hand-written before removal (Phase 2)
- [x] `.cloudcannon/postbuild` deleted (it held only `npm i` + `@bookshop/generate`)
- [x] `@bookshop/*` dependencies and the `dev`/`init`/`bookshop`/`bookshop-update` scripts removed; `package-lock.json` regenerated
- [x] Bookshop module imports removed; `component-library/` deleted
- [x] No Bookshop entries in `.gitignore` to remove
- [x] `_editables.text` added (regions use `data-type="text"`); no `data-type="block"` regions, so no `_editables.block`
- [x] Text and `<img>` parity against the Bookshop build — 25/25 pages identical

## Completeness checklist

Universal (Astro list, Hugo terms):

- [x] Editor enablement: `visual` first in `pages` and `blog` `_enabled_editors`
- [x] Census coverage: every row is implemented or has a justification
- [x] Array containers + items on every `range` rendered from front matter or data (numbers, slider, faq, hero_images, team_members, pricing_tier, feature, privacy, texts, nav items, dropdown, social, footer links)
- [x] Nested text/image regions inside every array item (social items have none: icon-only; the icon is a sidebar select)
- [x] Array path scope: relative paths inside items; `@data[...]` only on the component wrappers
- [x] Array container purity: every array wrapper holds only its items (footer `sections` deliberately not an array region; About hero images wrapped away from `.pattern`; privacy paragraph `texts` wrapped away from the `<h3>`)
- [x] Image regions on every content/data image (`data-prop-src` — all images are string paths)
- [x] Child component labels: hardcoded "Recent Blog" moved to data; remaining hardcoded strings are UI chrome ("minutes", "words", "© Copyright")
- [x] Shared partials backed by data: navbar (`@data[nav]`), footer (`@data[footer]`), recent heading (`@data[blog-post]`)
- [x] Data file completeness: all visible nav/footer values are in their data files
- [x] Cross-collection select wiring: n/a (categories render as links; author is a plain string)
- [x] `_inputs` presence audit: every `data-prop` key has an input (added blog `$.title`)
- [x] Schema-file seed audit: every wired field is in its `.cloudcannon/schemas/*.md`
- [x] Markdown body: `@content` on the blog post body wrapper
- [x] Conditional guards: optional regions (`prefix`, `suffix`, buttons, `nav_btn`) stay inside their existing `if`/`with`
- [x] Inline vs block text: markdown descriptions have inline-only toolbars → `data-type="text"`; no block hosts needed
- [x] `<template>` blueprints: none authored. Every sub-array sits inside a re-rendered component (block, navbar, footer, `legal/sections`), which renders new rows
- [x] Data file input config: every `data_config` file has a `file_config` entry

Page builder: array wrapper `data-component-key="_name"`; items carry `array-item` + `data-component` + `data-id`; widget sub-arrays wired; every `_name` resolves to a partial; build output contains all three attribute kinds.

Hugo items:

- [x] Component names resolve (all 16 `_name` values + `navbar`, `footer`, `legal/sections` exist under `layouts/partials/`)
- [x] Dispatcher in both `_default/single.html` and `_default/list.html`
- [x] Wrapper audit: dispatcher wrappers are direct children of `<body>` (block context, no `body > section` rules); array regions on Bootstrap `.row`s put `array-item` on the existing `.col-*` elements, so `.row > *` still matches; `<editable-component>` wraps `<header>`/`<footer>`/`<section class="privacy">`
- [x] Markdown pairing: one grep hit (`pricing/table.html` — `markdownify` is on the next line)
- [x] Body regions: `@content` on `blog/single.html`
- [x] Data-file regions: `nav`, `footer`, `blog-post` all in `data_config`
- [x] Empty arrays: `legal/sections` renders under `ENV_CLIENT` when `privacy` is empty; the page-builder wrapper is unconditional
- [x] Number fields: all bound numbers quoted, inputs `type: text`
- [x] Inline scripts: none in partials

## Other changes in this phase

- `navbar.html` and `footer.html` now take their data file as `.` (called as `partial "navbar.html" hugo.Data.nav` inside `<editable-component data-component="navbar" data-prop="@data[nav]">`), so the build and the editor pass the same shape. The navbar's active-link check uses `page`.
- `static/js/script.js`: count-up and magnific-popup are skipped when `window.inEditorMode` is set (count-up rewrites the counter text regions; the popup intercepts clicks on the editable video thumbnails).
- `_counter-up.scss`: `:nth-child(2|4)` → `:nth-of-type(2|4)` on `.counter-up-content` (array items).

## For the user to verify in CloudCannon

- `/` and `/blog/` open in the Visual Editor (`_index.md` URL collapse is unverified).
- FAQ question titles sit inside a Bootstrap collapse `<button>`; check that clicking one edits the text rather than only toggling the accordion.
- Counters show their real figures in the editor, meaning `window.inEditorMode` is set before `$(document).ready` runs.
- Navbar and footer edits from any page update `data/nav.yaml` / `data/footer.yaml`, and the navbar re-renders (dropdown toggle, nav button switch).
- Footer column headings (bound by index) edit the right column; footer link lists add/remove/reorder.
- Privacy page: add a "Bullet points" section and a paragraph; the scroll-spy nav updates.
- Pricing: toggling "Show a discounted price" re-renders the tier.

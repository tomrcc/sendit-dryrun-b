# Phase 2: Configuration

## Checklist (Hugo `configuration.md` § Verification checklist + the Astro items it inherits)

- [x] JSON schemas downloaded to `.cloudcannon/migration/` and gitignored (`.cloudcannon/migration/*.schema.json`)
- [x] `.cloudcannon/initial-site-settings.json` has `"ssg": "hugo"`, `build.output_path: public`, `build.hugo_version: "0.166.0"` (was the ignored `hugoVersion: 0.128.1`)
- [x] Every page-building collection has a `url` using `[full_slug]` — `pages: /[full_slug]/`, `blog: /blog/[full_slug]/`
- [x] `pages` excludes `blog/**`
- [x] No collection at `path: ''`
- [x] Every `_index.md` deliberately in or out — see below
- [ ] **Home page and a section list page opened in the Visual Editor, `_index.md` result recorded** — cannot be done here (no CloudCannon access). **User must verify** (see Handoff).
- [x] `permalinks`/`slug:`/`url:` overrides — none except `url: /blog/` on `content/blog/_index.md`, which equals Hugo's default path. Removed in Phase 3 so one pattern fits the collection; `url` is also hidden.
- [x] Every data file referenced by a region or live partial has `data_config` + `file_config`: `nav`, `footer` (both), `blog-tags` (select source + file_config). `meta.yaml` is head-only → `file_config` only.
- [x] Taxonomy `categories` is a top-level `multiselect` with `values: data.blog-tags`
- [x] `markdown` options match Goldmark defaults: `table: true`, `strikethrough: true`, `typographer: false`; Goldmark `unsafe` is false and no toolbar option emits raw HTML
- [x] Image inputs: all templates read `<img src="{{ .image_path }}">` from `static/` → `paths.static: static`, `paths.uploads: static/images`
- [x] `npx @cloudcannon/cli validate` passes (config, 16 structure-value files, 3 input files, initial-site-settings)
- [x] Every array has a structure linked with the full `_structures.<name>` path; every structure value has `preview` (+ `picker_preview` on blocks); nested objects have `type: object` + `preview.icon`
- [x] Enum-like fields are selects (`placer`, `seo.open_graph_type`, social `icon`); switches for `reversed`, `alternateStyle`, `no_top_pad`, `isActive`, `hasDiscount`, `enable_dropdown`, `enable_nav_btn`
- [x] Numbers shown as text (`numbers[].number`, `price`, `discountPrice`) are `type: text`; content quoting happens in Phase 3
- [x] Developer keys hidden: `_schema`, `_name`, `layout`, `type`, `aliases`, `url`, `build`, `cascade`; privacy `key` discriminator hidden
- [x] `add_options` restrict creation; blog `add_options` open in the Content Editor (drafts); page schemas have `new_preview_url`
- [x] `_enabled_editors` order: pages `[visual, data]`, blog `[visual, content, data]`, data `[data]`
- [x] `.cloudcannon/README.md` written
- [x] Snippets: no shortcodes in content and none defined → `_snippets_imports` removed (was `hugo: true`, which imported every built-in)

## Decisions

### Config file

The Bookshop-era `cloudcannon.config.yaml` was renamed to `cloudcannon.config.yml` and rewritten. The CLI baseline (`configure generate --dry-run`) was used as a reference only — running it for real would have written a second config file (`.yml`) beside the existing `.yaml`.

### Collections

| Key | Path | URL | Schemas | Notes |
| --- | --- | --- | --- | --- |
| `pages` | `content`, glob `**/*.md`, `!blog/**` | `/[full_slug]/` | `page_builder` (home, about, contact, feature, pricing), `legal` (privacy) | One collection, two schemas. `schema_key: _schema` |
| `blog` | `content/blog` | `/blog/[full_slug]/` | `default` (post), `blog_index` | `_index.md` **kept in** — its title/description drive the list hero. Not in `add_options` |
| `data` | `data`, glob `*.yaml` | `disable_url` | — | One collection, per-file `file_config` with preview icons |

The old config's `articles` key was renamed `blog` to match the Hugo section and the CLI's detection. No content referenced the key.

### `_index.md` URLs — unverified

`content/_index.md` (home) and `content/blog/_index.md` rely on `[full_slug]` collapsing `_index` to an empty string, as it does for `index`. The skill marks this **unverified**. Phase 3 moves `content/privacy/_index.md` to `content/privacy.md` so that the only `_index` files left are the two Hugo requires. **User must open `/` and `/blog/` in the Visual Editor and report the result.**

### Structures (Bookshop → hand-written)

The committed config had `_structures.content_blocks: { style: modal }` with no `values` — Bookshop generated them at build time. One structure-value file was written per `*.bookshop.yml` (16, including the unused `privacy/hero`), at `.cloudcannon/structures/content_blocks/<group>/<name>.cloudcannon.structure-value.yml`, collected with `values_from_glob`. `id_key: _name`; `value._name` is the Bookshop path (`home/hero`, …).

Bookshop → structure mapping corrections:

- `_inputs.image` in 7 `.bookshop.yml` files named no field (the fields are `image_path`) → rewritten as `image_path` with the same crop options.
- `_inputs.feature → _structures.features` in `pricing/table` → a `pricing_features` structure, linked from the `pricing_tiers` structure (the `feature` array is inside each tier).
- Blueprint numbers (`number: 200`, `price: 0`) quoted.
- `description` fields (rendered with `markdownify` inside `<p>`) → `type: markdown` with an inline-only toolbar, shared via `.cloudcannon/inputs/inline-markdown.cloudcannon.inputs.yml`. Button objects (`btn`, `link`) and split headings (`title` + `title_suffix`) are shared the same way.
- Page-level `description` (blog listing) renders as plain text → global `type: textarea`.

Shared sub-structures in the main config: `counter_numbers`, `hero_images`, `team_members`, `testimonials`, `faq_items`, `pricing_tiers`, `pricing_features`, `legal_sections` (two values keyed by `key: descriptive|bulletpoints`, `id_key: key`), `legal_texts`, `nav_items`, `nav_dropdown`, `social_links`, `footer_sections`, `footer_links`.

### Other

- `timezone` not set: the site config has no `timeZone` and every post date is UTC (`Z`), so CloudCannon's default `Etc/UTC` matches. (The skill says "otherwise ask" — no user to ask; recorded here.)
- Removed dead global inputs from the old config (`content`, `icon: [ph-user-square]`, `isActive` global, un-nested SEO keys like `page_description` that only matched because inputs match at any depth).
- `.cloudcannon/postbuild` (Bookshop generate) is removed in Phase 4 with the rest of Bookshop.
- `_editables.content` and `_editables.text` defined (Bookshop site had none). Content toolbar: headings h2–h4, lists, quote, link, image, table, strike, code.

## For the user to review

- `_index.md` URL collapse (above).
- The collection key changed from `articles` to `blog`; if the existing CloudCannon site has saved views or links to `collections/articles`, they need updating.

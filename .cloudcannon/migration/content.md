# Phase 3: Content

## Checklist (hugo/content.md)

- [x] Keep Hugo file conventions — no `_index.md` renamed in place, no bundles (none exist), YAML kept
- [x] Required fields present in every file (structure field-completeness script: all content blocks, nested arrays, legal sections, nav and footer items match their structure values)
- [x] `draft` present on every blog post (already)
- [x] Dates — one format (`YYYY-MM-DDT00:00:00Z`) across the blog (already)
- [x] Image paths — one convention (`/images/...` from `static/`) (already)
- [x] Numbers shown as text quoted
- [x] Routing overrides uniform — the only one (`url: /blog/`) removed
- [x] Field naming — keys match the case the content uses (`image_path`, `videoUrl`, `alternateStyle`, `placeHolder`, `hasDiscount` …; `data-prop` in Phase 4 uses the stored case)
- [x] Taxonomies top-level (`categories`)
- [x] Hardcoded layout content → front matter / data — "Recent Blog" heading moved to `data/blog-post.yaml`
- [x] Data files — all like-shaped lists already top-level arrays
- [x] Bookshop: `_bookshop_name` → `_name` in all 5 page files
- [x] Build with no new warnings; page count and per-page text + `<img>` count match the baseline

## Structural changes

| Change | Why |
| --- | --- |
| `component-library/components/<group>/<name>/<name>.hugo.html` → `layouts/partials/<group>/<name>.html` (16 files, contents unchanged) | `_name` is both the structure key and the partial path. Copied in this phase; `component-library/` is deleted in Phase 4 with the rest of Bookshop |
| `_default/list.html` and `_default/single.html`: Bookshop `bookshop_bindings` + `bookshop_partial "page"` → `range .Params.content_blocks` / `partial ._name .` | The rename of `_bookshop_name` breaks Bookshop's dispatcher, so the native dispatcher had to land in the same phase. Region attributes added in Phase 4 |
| `content/privacy/_index.md` → `content/privacy.md` with `layout: privacy`; `layouts/privacy/list.html` → `layouts/_default/privacy.html` | A one-file section becomes a `pages` entry (legal schema). It also stops relying on the unverified `_index` URL collapse. Output is still `/privacy/index.html` |
| Removed `url: /blog/` from `content/blog/_index.md` | Same as Hugo's default path; one URL pattern now fits the whole `blog` collection |
| `_schema:` added — `page_builder` (5 pages), `legal` (privacy), `blog_index` (blog list), `default` (5 posts that lacked it) | Explicit schema matching |
| Quoted `numbers[].number` (8), `price` / `discountPrice` (6) | Text regions reject bare numbers |
| Added `prefix: ''` / `suffix: ''` to 6 counter items, `no_top_pad: false` to the Feature page header | Structure field completeness (template checks `if .prefix` / `with .no_top_pad`, so empty values render the same) |
| New `data/blog-post.yaml` (`recent_heading: Recent Blog`); `blog/single.html` reads it via `index hugo.Data "blog-post"` | The heading is visible on every post, so it must be editable; `data_config` + `file_config` added |
| Deleted root `schemas/` (`page.html`, `post.md`) | Replaced by `.cloudcannon/schemas/` |

## Parity

Baseline = the pre-migration commit plus the Hugo 0.166 upgrade fixes (`css.Sass`, `hugo.Data`, `locale`) — the untouched commit doesn't build on 0.166. Phase 3 build: 25/25 HTML pages present, **0 pages differ** in visible text or `<img>` count. The remaining `.Site.Data` deprecation warning comes from the Bookshop module (removed in Phase 4).

## Left as-is (judgment)

- Blog `author` stays a free-text string: the templates only print the name (no bio or avatar), and 5 names cover 6 posts.
- `404.html` stays a hardcoded layout (no file to write back to).
- Blog meta labels ("minutes", "words") and pagination arrows are UI chrome, not content.

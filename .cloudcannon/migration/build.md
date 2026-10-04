# Phase 5: Build and test

## Build verification checklist (hugo/build.md)

| Check | Result |
| --- | --- |
| **Clean build** — `rm -rf public resources/_gen && npm run build` | ✅ Pass. 27 pages, 0 warnings, 0 errors (Hugo 0.166.0 extended) |
| **Page parity** — page lists vs the pre-migration commit | ⚠️ The pre-migration commit **doesn't build** on Hugo 0.166 (`resources.ToCSS` removed; build.md's `hugo -s /tmp/before` step assumes it does). Compared against the pre-migration commit **plus the upgrade-only fixes** instead: page lists identical (25 HTML files) |
| **Text and image parity** — visible text and `<img>` count per page | ✅ 25/25 pages identical (script ignores tags and whitespace; `<editable-*>` wrappers are stripped without adding spaces) |
| **Regions in the output** — `grep -roE 'data-editable="[a-z-]+"' public` | ✅ `array-item` 505, `text` 268, `array` 147, `image` 82; plus 404 `<editable-text>`, 39 `<editable-component>`. `data-component=` on every page-builder item (16 block names), `navbar`/`footer` on every non-alias page, `legal/sections` on `/privacy/` |
| **Editor-mode build** — `hugo --config config.toml,envclient.yaml` with `params.ENV_CLIENT: true` | ✅ `Processed images │ 0`; 0 pages differ from the normal build in text or `<img>` count |
| **Renderer published** — `public/_cloudcannon/` | ✅ `hugo_renderer.wasm.d5ed9d7c….gz`, `live-editing.7c4e6e39….js`, `hugo-worker.7d7654fa….js` |
| **Bundle contents** — every `data-component` partial is a `"layouts/partials/<name>.html"` key in `live-editing.*.js` | ✅ All 19 (16 blocks + `navbar`, `footer`, `legal/sections`) |
| **Config validates** — `npx @cloudcannon/cli validate` | ✅ Exit 0 — 21 files valid: `cloudcannon.config.yml`, 16 structure-value files, 3 input files, `initial-site-settings.json` |
| **Search index** | n/a — no Pagefind |

## CloudCannon build settings

`.cloudcannon/initial-site-settings.json`: `ssg: hugo`, `install_command: npm i`, `build_command: npm run build`, `output_path: public`, `hugo_version: 0.166.0`, `HUGO_CACHEDIR` env and `node_modules/,resources/,.hugo_cache/` preserved. There are no hook files: `.cloudcannon/postbuild` was Bookshop-only and was deleted.

**Existing site:** `initial-site-settings.json` only applies when a site is first created. If this repo is already connected to a CloudCannon site, set Hugo to **0.166.0** in **Site Settings → Builds**. The old file's `hugoVersion` key was ignored, so that site is building on CloudCannon's default Hugo. That setting was not changed here.

## Handoff

### What changed

Bookshop is replaced by CloudCannon editable regions. Components moved to `layouts/partials/<group>/<name>.html` and the pages keep their existing `content_blocks`, now keyed by `_name`. The config was rewritten with hand-written structures for every block and sub-list. Header, footer and the blog "Recent" heading are editable data files. The privacy page moved into the Pages collection, and Hugo was upgraded to 0.166.

### Already checked locally (no need to repeat)

Clean build, output parity (text + images on all 25 pages), editor-mode build, renderer bundle, region grep, config validation.

### Please check in CloudCannon

- [ ] `npm run build` succeeds on CloudCannon (build log) with Hugo 0.166.0
- [ ] `/` (home) and `/blog/` open in the Visual Editor — the `_index.md` URL mapping is unverified
- [ ] Home, About, Pricing: click headings, descriptions, buttons and images to edit them; add, reorder and remove a section with the page-builder controls
- [ ] Counters show their real figures and are editable (count-up animation is disabled in the editor)
- [ ] Pricing: edit a price and a feature, toggle "Show a discounted price" and "Highlight this tier"
- [ ] FAQ: clicking a question edits its text (it sits inside the accordion button)
- [ ] Privacy: edit a heading and a paragraph, add a bullet-points section
- [ ] Blog post: edit title, author, featured image and body; the "Recent Blog" heading edits `data/blog-post.yaml`
- [ ] Navigation and footer: edit a menu label and a footer link from any page, then save and confirm the change lands in `data/nav.yaml` / `data/footer.yaml`
- [ ] "+ Add" in Pages offers "Page builder page" and "Legal page"; "+ Add" in Blog opens a new draft post in the Content Editor
- [ ] Saved edits commit only the edited values (review one git diff per file type)

When you've run these, reply with any failures and the page URL plus what you clicked.

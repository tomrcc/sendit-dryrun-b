# Dry run B: Sendit Hugo, French, blog posts as files (setup B)

Request: "Make this site multilingual with French. We want blog posts translated as separate files our editors can edit, not as strings."

Path taken: `make-site-multilingual/SKILL.md` → `hugo/overview.md` § Choose a setup → **B (hybrid)** → `setup.md` phases with `hugo/hybrid.md` → `translate-site` Part 2 (content files) and Part 1 (locale JSON).

## What I did, by doc section

1. **SKILL.md → hugo/overview.md § Choose a setup.** No `languages` block and the site owns its layouts, which points to C. The user asked for per-file body translation, so I chose **B** and stated the cost (two-step generate, a French `_index.md` per list page, no `{{ i18n }}`, the fix script).
2. **setup.md Phase 1.** Output dir is `public/`. Default language at root (user). No Bookshop.
3. **Phase 2.** Ran `npm install`, then `npx rosey-cloudcannon-connector init --yes --locales fr --build-dir public`. It installed `rosey` and `rosey-cloudcannon-connector`, wrote `.cloudcannon/postbuild`, and added the `locales` collection and `data_config.locales_fr`. I re-read the config as the doc says.
4. **hybrid.md § Hugo config.** Added `defaultContentLanguage`, `defaultContentLanguageInSubdir = false`, `disableDefaultLanguageRedirect`, `[languages.en|fr]` with `contentDir`, and `params.locales` (TOML here, not YAML).
5. **Phase 3 / tagging.md + overview § Rules for B and C.**
   - `baseof.html`: root from `.Path` on a new `<main>`, `<html lang="{{ site.Language.Lang }}">`, `<title>` keyed only when content keys are allowed, a `data-rcc` boundary around nav, main and footer, `data-rcc-exclude` on pages with a French file, the RCC import from `/_rcc/client.mjs`, and the Phase 9 picker script.
   - `meta.html`: a page's own description keyed `{root}:page_description` (only where content keys are allowed). The site-wide fallback from `data/meta.yaml` uses one shared `site:page_description` key (§3h).
   - Nav and footer: `data-rosey-ns="nav"` / `"footer"`, content-as-key on every `editable-text`, and `attrs-explicit` on the toggle's `aria-label`.
   - Page-builder partials: block `data-rosey-ns="{{ ._uuid }}"`, nested array items `data-rosey-ns="{{ ._uuid }}"`, keys derived from `data-prop` with `.` → `_` (§3f). Names, numbers, prices, phone and email are left unkeyed. Everything is gated by a helper (below).
   - Blog: post title and `@content` (with `data-type="block"`) gated. `minutes` and `words` are keyed under root `blog-post`. "Recent Blog" uses root `blog-post`. Card titles use root `""` with key `{path}:title`, so they share the post's own title key. Category chips go through one label helper with root `categories` (§3i). Category links now use `relURL` instead of `absURL` (overview § Links). Pagination arrows got `aria-label` + `attrs-explicit`.
   - 404 strings are keyed.
6. **_uuid seeding (§3g).** A line-level, idempotent pass over the front matter of `content/*.md` added a `_uuid` to every map list item. I added an empty `_uuid:` to every structure default (`*.structure-value.yml`, including nested default items, and the inline `_structures` that hold text). I added a global hidden `_uuid` input with `instance_value: UUID`.
7. **hybrid.md § A real `_index.md`.** Seeded `content_fr/_index.md`, `content_fr/blog/_index.md`, `content_fr/about.md` and two posts (`email-delivery-tips`, `best-email-sending-services`) as copies.
8. **translate-site Part 2.** Ran `prepare-content-translation.mjs` for `content`→`content_fr` and `content/blog`→`content_fr/blog`. The blog posts went through prepare → translate → merge cleanly. Home and About are page-builder pages the script can't parse (friction #1), so I translated them with the **manual fallback**, along with the blog index description.
9. **hybrid.md § Postbuild.** Replaced init's postbuild with the hybrid one: generate from a copy without `fr/`, write-locales, install-client, rosey build, then `fix-rosey-pages.mjs --pager-segment page`.
10. **Phase 8 step 9.** Added `pages_fr` (`content_fr`, `/fr/[full_slug]/`) and `blog_fr` (`content_fr/blog`, `/fr/blog/[full_slug]/`) as copies of the English collections, with French `new_preview_url`s. I also added them, and `locales`, to `collection_groups`.
11. **Phase 9.** Added a `locale-picker.html` partial (overview's code) to the navbar. It is guarded by `site.Params.ENV_CLIENT`, because the navbar is re-rendered by the editor's in-browser Hugo.
12. **translate-site Part 1.** Ran prepare → translate → merge twice. I filled nav, footer, buttons (`btn_text`, submit), 404, category labels, `minutes`/`words`/"Recent Blog", the site description, and the term/taxonomy page titles. Page content keys on Rosey-only pages (feature, pricing, contact, privacy, the four untranslated posts) are left English, as the user asked.
13. **Extras the docs didn't ask for, but a user needs:** `CLOUDCANNON_SYNC_PATHS=/rosey/` in `initial-site-settings.json` (from init's printed output), and `_untranslated_site/` + `_rosey_source/` in `.gitignore`.

Defaults I picked where the docs or the user didn't say:

- Home and blog index translated as files. The hybrid MUST forces these files to exist, and leaving them as English copies would show English on `/fr/`.
- Page content on Rosey-generated pages left untranslated. The user only asked for UI strings.
- Dates left as `:date_long`, so they are French on Hugo-built pages and English on Rosey copies (hybrid.md accepts this).
- Junk `alt` values (`Nav-Logo`, `blog-thumb`) and icon-only social links deliberately not keyed.
- `rosey/base.json` not committed (no-commit rule). Phase 6b says to commit it.

## Build / postbuild results

`PATH=node_modules/.bin:../.bin:$PATH npm run build` → Hugo 0.166, EN 28 pages / FR 17 pages, no errors or warnings.
`bash .cloudcannon/postbuild` →

```
Rosey: Finished
RCC: Wrote rosey/locales/fr.json — 151 keys
RCC: Wrote locale manifest → public/_rcc/locales.json
RCC: Wrote client → public/_rcc/client.mjs
Rosey: Finished
fix-rosey-pages [fr]: 10 generated pages, 0 paginator pages deleted, 9 added to fr/sitemap.xml
```

I also ran an editor-mode build (`ENV_CLIENT: true`, per cloudcannon-visual-editing) with no errors. The picker is absent in that build. `public/_cloudcannon/live-editing.*.js` bundles `rosey-content.html`, `locale-picker.html`, `category-*.html` and the config with both `contentDir`s.

## Verified in `public/` after the postbuild

- **6a on `rosey/base.json`:** 151 keys. None dotted, none with `undefined` or an empty segment, no `<svg`/`<!--`/`editable-text` in any original, no key with an `fr/` page. `_rcc/locales.json` = `{"locales":["fr"]}`, and `_rcc/client.mjs` exists.
- **No content keys for file-translated pages:** `index`, `about`, `blog` and the two translated posts have no keys in `base.json`.
- **`/fr/blog/email-delivery-tips/` (Hugo-built):** `lang="fr"`, French `<title>`, French h2, `7 mars 2022`, French body, "mots", "Articles récents". Recent cards list only the two French posts. Nav and footer are French, links point to `/fr/…`, canonical is `…/fr/blog/email-delivery-tips/`. Two `hreflang` links. Picker links point to `/blog/email-delivery-tips/` and `/fr/blog/email-delivery-tips/`. `data-rcc-exclude="fr"`.
- **`/fr/blog/what-are-the-best-email-clients/` (Rosey-generated):** English body and title (untranslated, as intended), UI strings French, canonical fixed to `/fr/…`, keyed `<title>`, `data-rcc` with no exclude. One recent card ("Best email sending services") shows the English title: see friction #5.
- **`/fr/` home and `/fr/about/`:** every visible string is French, apart from names and numbers.
- **`/fr/blog/`:** French title and description. It lists only the two French posts (expected, per hybrid.md).
- **`/fr/contact/` (Rosey):** UI French, content English, canonical `/fr/contact/`.
- **`/fr/categories/email/` (Hugo-built term):** h1 `E-mail` and title `E-mail | Sendit`, but only after fixing friction #3. **`/fr/categories/marketing/`** is a Rosey copy listing English posts (expected).
- **`/fr/404.html`:** title `404 Page introuvable | Sendit` (after friction #3) and French body.
- **hybrid Verify:** no `{{ i18n }}`/`{{ T }}`, one hreflang set per page, no `/fr/en/`, no stray `/fr/blog/page/N/`, and `/fr/contact/` canonical is the French URL. English pages keep `lang="en"` and no Rosey injection.

## Friction table

| # | Doc file + section | What it said | What actually happened / what was missing | What I did |
| - | ------------------ | ------------ | ----------------------------------------- | ---------- |
| 1 | `translate-site/content-directories.md` Phase 2.1 / 2.3 (scripts) | Run `prepare-content-translation.mjs --source-dir content --locale-dir content_fr`. It extracts translatable front matter "by dot-notation path" | The parser skips every `- ` line and flattens array items. On page-builder pages (`content_blocks`), every block's `title` collapses to one `content_blocks.title` (last one wins), and nested arrays merge the same way. Folded scalars (`description: >-`) come out as the literal string `">-"`, marked translatable. This also hit the plain blog `_index.md`. The merge script replaces only the key line, so translating a `>-` field would leave the old continuation lines in place, and YAML would silently concatenate French + English. hybrid.md forces `content_fr/_index.md` for the home page, so a Sendit-style site always hits this | Translated home, about and the blog index description by hand (manual fallback). Used the scripts only for the two posts |
| 2 | `hugo/hybrid.md` § A real `_index.md` for every French list page; `overview.md` "You give up" table | MUST add `content_fr/_index.md` and a section `_index.md`. Cost described as "a French `_index.md` per list page" | Nothing says this turns the **home page** (here a page-builder page) into a file-translated page. Its content leaves Rosey: it loses inline RCC editing, gets `data-rcc-exclude`, and must be translated as a file before `/fr/` stops showing English. The user asked only for About and two posts. No guidance on whether to seed a full copy or a minimal file | Seeded full copies and translated them. Recorded as a decision the user should have been told about |
| 3 | `hugo/hybrid.md` § Content keys only where Hugo has no translation; § Hide the RCC switcher | `$roseyContent := and (eq site.Language.Lang default) (not .IsTranslated)`; `data-rcc-exclude` when `.IsTranslated` | Hugo builds 404, taxonomy lists and term pages in every language **without a content file**, so `.IsTranslated` is true. Following the doc, `/fr/404.html` kept `404 Page not found \| Sendit` and `/fr/categories/email/` kept `Email \| Sendit` with no key to translate them. The switcher was also excluded on those pages. On French Hugo-built pages the doc's expression is always false, so these pages can never carry keys | Wrote a `rosey-content.html` helper: keys are suppressed only when a **non-default-language translation has a `.File`**, in both languages. Used it for `data-rcc-exclude` too. Re-verified 404 and term titles are French |
| 4 | `hugo/hybrid.md` § Content keys…; `tagging.md` §3g | Shows `$roseyContent` in the page template only | Sendit's content lives in component partials that CloudCannon re-renders with block data as context (not the page). Nothing says how those partials should get the flag. The answer is `page`, per cloudcannon-visual-editing. It is unverified whether `page.AllTranslations` / `hugo.Sites` work in the editor's in-browser Hugo | Partials call `partial "rosey-content.html" page`. Added to manual checks |
| 5 | `hugo/hybrid.md` (whole file) | Rules cover a page's own content keys | Nothing covers **cross-page references** to a file-translated page from a Rosey-generated page: recent-post and listing cards, related links. Gating a card by the card post's translation status leaves its English title on `/fr/` Rosey pages ("Best email sending services" on `/fr/blog/what-are-the-best-email-clients/`). Keying it anyway creates a second source of truth beside `content_fr/` | Gated cards by the card post's helper result. Left the wart and noted it |
| 6 | `tagging.md` §3g Seeding `_uuid` | Edit line-level and anchor on the discriminator. "Expect the anchor to miss things" (counters, …). "Cover structure defaults and data files too" | No method for nested object arrays without a discriminator (counter numbers, FAQ items, pricing tiers/features, legal texts, testimonials). These are exactly Sendit's text-bearing arrays. "Cover structure defaults" doesn't say that defaults need an **empty** `_uuid:` (for `instance_value`), not a seeded value. It is also unclear whether `instance_value` fills nested items inside a newly added block's default. Only pointer: the Astro starter | Wrote my own line-level seeder for all map list items in page front matter, and added empty `_uuid:` to structure defaults, including nested defaults |
| 7 | `tagging.md` §3g "put rosey attributes inside each item's component" | Make each array item its own registered component | In Hugo editable-regions, nested arrays are rendered inline in the block partial's loop, with no `data-component` of their own. The doc's only example is Astro, so it's unclear whether putting `data-rosey-ns` on a nested `array-item` inside a re-rendered block component is compliant | Put the ns on the nested `array-item` element inside the block partial. Added to manual checks (add/reorder a FAQ item) |
| 8 | `setup.md` Phase 2 / Phase 5 | Nothing about syncing build output | `init` prints "1. Sync paths: set `CLOUDCANNON_SYNC_PATHS=/rosey/`" as its **first** next step. No skill doc mentions it. Without it, keys added by builds never reach the repo's `rosey/locales/fr.json`, which the Locales collection and the RCC read | Added it to `initial-site-settings.json`. It must also be set in site settings for an existing site |
| 9 | `setup.md` Phase 2 ("CloudCannon config … already done"), Phase 5d; `hybrid.md` Phase 8 step 9 | init configures CloudCannon; add `pages_fr` / `blog_fr` | Sendit defines `collection_groups`. `init` added `locales` without adding it to a group, and hybrid step 9 doesn't mention groups (native-multilingual.md does). New collections may be missing from the sidebar | Added `pages_fr`, `blog_fr` (a "Français" group) and `locales` to `collection_groups` |
| 10 | `setup.md` Phase 2 | "If you ran init, the postbuild pipeline (Phase 4) and CloudCannon config (Phase 5) are already done — skip to Phase 3, then Phase 6" | Phase 5a (RCC import) and 5b (`data-rcc` boundary) are template edits init only **prints**. Following the skip literally leaves the RCC unloaded. Also, init's postbuild has `mv ./public ./_untranslated_site` with no `rm -rf` first (Phase 4's template has it), so a second local run nests `public` inside the old directory | Did 5a/5b by hand. Hybrid's postbuild replaces init's, so B avoids the `mv` bug, but A/C users who "verify and move on" won't |
| 11 | `hugo/overview.md` § Visitor-Facing Locale Picker | Partial using `.AllTranslations` and `hugo.Sites.Default`; "place it in both desktop and mobile nav" | On a CloudCannon site the nav is usually a re-rendered editable component, so the picker partial runs in the editor's Hugo too. Nothing says to guard it, though it's hidden in the editor anyway | Wrapped the call in `{{ if not site.Params.ENV_CLIENT }}` |
| 12 | `translate-site/locale-files.md` § Intentional non-translations | Set `value` equal to `original` for brand names and identical words | The classifier treats `value === original` as untranslated. After my merge of 39 entries, prepare reported only 31 current. `minutes`, `Blog`, `Contact`, `Marketing`, `© Copyright` and others are offered for translation again on every run | Left as-is. Noted |
| 13 | (none) | — | No doc says to gitignore `_untranslated_site/` and `_rosey_source/`, which the postbuild leaves behind locally | Added both to `.gitignore` |
| 14 | `hugo/overview.md` Setup C table, Phase 3 | `<html lang="en">` | hybrid.md doesn't say that in B the attribute must follow the language (`site.Language.Lang`). A literal `en` would mislabel Hugo-built French pages until Rosey rewrites them | Used `site.Language.Lang` |
| 15 | `translate-site/content-directories.md` Phase 2.1 | Manifest written next to the locale dir | With `--locale-dir content_fr`, the manifest lands in the project root. With `content_fr/blog`, it lands **inside Hugo's `content_fr/`**. Both have the same filename `.translation-task-fr-content.json`. Harmless, but surprising | None needed. The merge deletes it |

Smooth parts: the choose-a-setup questions, root from `.Path`, the hybrid Hugo config (no `/fr/en/`), the hybrid postbuild and fix script (canonical and sitemap correct first time), Rosey link rewriting on both kinds of `/fr/` page, the Part 1 scripts, and the Part 2 scripts on plain blog posts.

## Manual checks for CloudCannon

1. Site settings: `CLOUDCANNON_SYNC_PATHS=/rosey/` is set (initial settings apply only to new sites). After the first build, `rosey/base.json` and `rosey/locales/fr.json` appear in the repo.
2. Sidebar: the "Français" group shows Pages (Français) and Blog (Français), and Site shows Locales.
3. Open `content_fr/blog/email-delivery-tips.md` and `content_fr/about.md` in the Visual Editor. They load at `/fr/…`, show **no** RCC locale switcher, and an edit (body, a block title) saves to the French file and survives a reload.
4. Open a Rosey-only page (e.g. `/pricing/` or `/blog/what-are-the-best-email-clients/`). The RCC switcher appears. Switch to FR, edit a nav item and a FAQ answer, reload, and confirm both persisted to `rosey/locales/fr.json`.
5. On `/about/` and `/` (file-translated), confirm there is no switcher. On `/404` and a category page, confirm the switcher appears.
6. Edit a block in the Visual Editor on `/pricing/` and check the block re-renders without a "Failed to render Hugo component" error, and still carries its `data-rosey`/`data-rosey-ns` attributes. This tests whether `page.AllTranslations` / `hugo.Sites.Default` work in the in-browser Hugo.
7. Add a new FAQ item and a new pricing feature. Check each gets a fresh `_uuid` (nested `instance_value`), and that a reorder doesn't swap translations.
8. Edit the nav (`data/nav.yaml`) in the Visual Editor. The navbar re-renders, and the language picker is hidden in the editor.
9. Add a new French blog post via Blog (Français) → it gets the `default` schema and opens at `/fr/blog/<slug>/`.
10. Locales collection: values edit as rich text, and head-only keys (`*:page_title`, `site:page_description`) are reachable there.
11. On the live site: the language picker's look in the navbar (it has no styling) and the active-state highlight on a Rosey page.

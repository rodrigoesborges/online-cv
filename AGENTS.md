# AGENTS.md

Jekyll static site for the personal CV/résumé of Rodrigo Borges (content in **pt-BR**), forked from the `sharu725/online-cv` theme (Orbit, by Xiaoying Riley). Deployed on GitHub Pages under the custom domain `rodrigo.borges.net.br` (the `CNAME` file — do not delete).

## Branches (important)

- **`gh-pages`** — the working and deployed branch. All content edits go here. GitHub Pages builds the site from this branch (Jekyll source, not pre-built output).
- **`master`** — mirrors the upstream theme; used only for pulling upstream updates. Do not commit local content changes there.

## Commands

```sh
bundle install                # Ruby deps (jekyll 3.9.5 pinned via github-pages gem, 231)
bundle exec jekyll serve      # dev server at http://localhost:4000
bundle exec jekyll build      # production build into _site/ (gitignored)
docker-compose up             # alternative: jekyll/builder:4.0 container, port 4000,
                              # --watch --force_polling --livereload
```

There are no tests and no linter. Verification = the site builds without errors and renders correctly at `/` and `/print`.

## Architecture

Two pages compose the site by including section partials from `_includes/`:

- `index.html` (web view) → career-profile, education (conditionally), skills, experiences, projects, publications, recommendations
- `print.html` (permalink `/print`, layout `print`) → adds certifications and oss-contributions, and orders skills after experiences

**All content lives in `_data/data.yml`** — the single source of truth. Almost never touch HTML to change content.

### Data flow

Each include reads its top-level key via `site.data.data.<key>` and is wrapped in `{% if section %}`. Commenting out or removing a top-level key in `data.yml` removes that section from the page (this is how sections are toggled).

Section data shapes (not obvious from include names):

| Key | List field | Item fields |
|---|---|---|
| `experiences` | `info` | `role`, `company`, `time`, `details` (Markdown) |
| `projects` | `assignments` | `title`, `link`, `tagline`, `technologies` |
| `publications` | `papers` | `title`, `link`, `authors`; venue goes in `book`, `review`, **or** `conference` (all three render identically) |
| `recommendations` | `list` | `person`, `role`, `email`, `phone`, `recommendation` (Markdown) |
| `oss` | `contributions` | `title`, `link`, `tagline` |
| `skills` | `toolset` | `name`, `level` (e.g. `95%` — drives the progress bar width) |
| `sidebar.idiomas` | `info` | `idiom`, `level` — **key is `idiomas`, not `languages`** (local customization of `language.html`) |
| `certifications` | `list` | include exists but key is absent from data (renders nothing) |

### Education is dual-mode

`sidebar.education: true` renders a compact block in the sidebar; `false` renders it as a full main-column section. The guard lives **inside** `_includes/education.html`, and both `sidebar.html` and `index.html` include it — do not add another guard in the callers.

### Sidebar / contact

`_includes/contact.html` renders each social link only if its `sidebar.<key>` exists. Custom links added locally vs upstream: `moodle` and `blog` (auto-prefixed with `https://`). Note the `pdf` sidebar link points to an external short URL, not the generated PDF.

## Styling / theming

- Skin is chosen by `theme_skin` in `_config.yml` → imports `_sass/skins/_<skin>.scss` from `assets/css/main.scss`. Available skins: blue, turquoise, green, berry, orange, ceramic, teal, oceanstale (current: **berry**).
- `assets/css/main.scss` is the Sass source; Jekyll compiles it to `assets/css/main.css` (what `head.html` links). Never edit a `main.css`.
- `sidebar.position` (left/right in `data.yml`) is read by **Liquid inside `main.scss`** to set flex-order variables — sidebar position is decided at CSS compile time.
- `_layouts/compress.html` (HTML minification) exists but is **disabled**; front-matter comments in `_layouts/default.html` and `_layouts/print.html` show where to enable it.
- `assets/plugins/` (Font Awesome, Bootstrap) is vendored third-party code — never hand-edit.

## PDF / print

`assets/js/pdf-generator.js` (loaded only on the default layout) powers two buttons:
- **Print**: opens `/print` in a new window and calls `print()`. Beware: it defines a global `print()` that shadows `window.print`.
- **PDF**: fetches `/print` client-side and runs html2pdf.js (CDN, in `head.html`) to save `<Name>_Resume.pdf`, where the name comes from the `.name` DOM element.

## Gotchas

- **`_data/data.yml` syntax errors break the whole build** (warning at the top of the file). YAML block-scalar indentation (`details: |`) directly affects rendered Markdown.
- **`_config.yml` changes (e.g. `theme_skin`) require restarting `jekyll serve`** — config is not hot-reloaded.
- HTML entities are used intentionally inside YAML strings (e.g. `Habilidades &amp; Proficiência`, `Linhas de Pesquisa &amp; Interesses`) because titles are injected raw into HTML.
- Publication entries without URLs use `link: "#"` rather than omitting the field.
- `baseurl` is `'/'` and every asset/include references `{{ site.baseurl }}` — keep it consistent if the site is ever served from a subpath.
- Analytics (`_includes/analytics.html`) only renders if `site.analytics` is set in `_config.yml` (currently commented out).
- Footer text (including theme attribution) comes from the `footer` key in `data.yml`.

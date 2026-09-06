# Moonwalk migration design

Date: 2026-09-06
Repo: 0jonjo/0jonjo.github.io (user site, served at https://0jonjo.github.io/)

## Goal

Replace the Gradfolio theme with Moonwalk, move the build from GitHub Pages
legacy mode to GitHub Actions, add lightweight language separation (pt/en),
and remove dead or heavy dependencies. Keep every existing URL working.

## Current state

- Jekyll 4.4.1 in Gemfile, but Pages legacy build uses Jekyll 3.10 and ignores
  the Gemfile. Local build is broken (missing native gems, Ruby 3.4 default
  gems removed, `url:` empty in `_config.yml`).
- 61 posts (36 pt, 25 en). English posts are marked only by the tag `english`.
- Theme: Gradfolio (2021 copy). ~900 lines SCSS, FontAwesome 5 loaded via
  remote JS on every page, MathJax 2.7 on every page (3 posts use it),
  ga-lite with empty tracking id, Disqus include never enabled.
- Collections: `_projects` (3 items). Pages: `/archive/`, `/tags/`,
  `/projects/`, `/blog/` (paginated, 4 per page), `/404.html`.
- Scheduled post (2026-09-07) does not publish on its own because legacy
  build only runs on push.

## Design

### 1. Theme

Copy Moonwalk (https://github.com/abhinavs/moonwalk) `_layouts`, `_includes`,
`assets` into this repo. Moonwalk is a template repo, not a gem theme, so it
is vendored. Gradfolio layouts, includes and SCSS are deleted. Work on branch
`moonwalk`, merged via PR.

Moonwalk settings used: light theme by default (dark toggle stays available
because it ships with the theme, it is not a requirement). Social links:
GitHub, LinkedIn, email. Favicons and `site.webmanifest` kept.

### 2. URLs

Unchanged:

- `permalink: /blog/:year/:title/`
- `paginate_path: /blog/page:num/`, `paginate: 4`
- `/archive/`, `/tags/`, `/projects/`, `/projects/:name/`, `/404.html`

Verification: build old and new site, diff the sorted list of generated
`.html` paths. New paths may appear (feeds, sitemap, `/en/`, `/pt/`). No old
path may disappear.

### 3. Deploy via GitHub Actions

`.github/workflows/pages.yml`:

- Triggers: `push` on `main`, `schedule` cron `0 9 * * *` (09:00 UTC daily,
  so future-dated posts publish on their date), `workflow_dispatch`.
- Steps: checkout, `ruby/setup-ruby` with bundler cache, `bundle exec jekyll
  build` (default `future: false`), `actions/upload-pages-artifact`,
  `actions/deploy-pages`.
- Permissions: `pages: write`, `id-token: write`.

Repo setting change: Pages Source from "Deploy from a branch" to "GitHub
Actions". Done via `gh api` or Settings UI at merge time, with user's ok.

### 4. Language separation

Front matter: every post gets `lang: pt` or `lang: en`. Migration script:
posts whose `tags` include `english` become `lang: en` and the `english` tag
is removed; all others become `lang: pt`.

Presentation:

- Post list (home and `/blog/`): filter buttons `All · EN · PT` above the
  list. Small inline JS (~15 lines) toggles `[data-lang]` items and stores
  the choice in `localStorage`. Without JS the list shows all posts.
- Each post entry and each post header shows a language badge.
- Pages `/en/` and `/pt/` list only that language (no JS needed) and expose
  feeds `/en/feed.xml` and `/pt/feed.xml` via `jekyll-feed` collections or a
  filtered feed template.
- `<html lang="{{ page.lang | default: site.lang }}">`; `site.lang: pt`.
- Tags page: same `All · EN · PT` filter applied to tag lists; all tags
  shown, not only those with more than 3 posts.
- Site chrome (nav, footer, buttons) stays in English.

### 5. Cleanup

Removed:

- FontAwesome remote script (Moonwalk ships inline SVG icons).
- MathJax global include. Becomes conditional: `{% if page.math %}` in the
  post layout. The 3 posts using math get `math: true`.
- `_includes/analytics.html` (ga-lite, empty id), `_includes/disqus.html`,
  `_includes/youtubePlayer.html` if unused after check, `_layouts/compress.html`.
- Gradfolio SCSS and vendored fonts.

Added:

- `jekyll-feed`, `jekyll-sitemap` to Gemfile and `plugins`.
- `url: https://0jonjo.github.io` in `_config.yml` (fixes `URI::InvalidURIError`
  from `jekyll-seo-tag`).
- Gemfile: `csv`, `base64`, `bigdecimal`, `logger` (Ruby 3.4 no longer bundles
  them). Keep `jekyll-seo-tag`, `jekyll-paginate`, `jekyll-email-protect`,
  `jekyll-target-blank`.

Tag normalization: a mapping applied to all posts, reviewed by the user before
running. Known cases: `progamacao`/`programação` → `programming`,
`refatoracao` → `refactoring`, `projeto` → `project`. Tags stay lowercase,
single-word, English.

### 6. Projects

`_projects` collection kept. Rendered as a simple list on `/projects/` using
Moonwalk's list styling (title, description, link). Individual project pages
keep `/projects/:name/`.

### 7. Verification before merge

- `bundle exec jekyll build` succeeds locally with no warnings.
- `htmlproofer` (via `html-proofer` gem, dev group) passes on internal links
  and images.
- URL diff (section 2) shows no removed paths.
- Manual check in Chrome: home, one pt post, one en post, `/tags/`, `/en/`,
  `/projects/`, `/blog/page2/`, 404.
- After merge: Actions run green, site live, then trigger `workflow_dispatch`
  once and confirm the 2026-09-07 post appears on or after that date.

## Out of scope

Visual redesign beyond Moonwalk defaults, comments, analytics, translating
existing posts, Bridgetown or non-Ruby generators.

# Moonwalk Migration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace Gradfolio with a vendored Moonwalk theme, deploy via GitHub Actions with a daily cron, add pt/en separation, remove dead dependencies, keep every existing post URL.

**Architecture:** Jekyll 4 site. Moonwalk's `_layouts`, `_includes`, `_sass` are copied into the repo (no gem, no remote_theme) so every file is editable. Language is a `lang` front-matter key on each post; lists render `data-lang` items with a badge, a ~20-line inline script filters them and remembers the choice. Deploy is one workflow: `jekyll build`, upload artifact, deploy to Pages.

**Tech Stack:** Ruby 3.4.4, Jekyll 4.4, jekyll-feed, jekyll-sitemap, jekyll-seo-tag, jekyll-paginate, jekyll-email-protect, jekyll-target-blank, html-proofer (dev), GitHub Actions (`actions/upload-pages-artifact@v3`, `actions/deploy-pages@v4`).

Spec: `docs/superpowers/specs/2026-09-06-moonwalk-migration-design.md`

Deviations from the spec, decided while reading the sources:
- No post uses math. MathJax is removed entirely, no `math: true` include.
- Projects: the `_projects` collection (Alljobs, Calcpace gem, Woche) is dropped. The home page gets a one-paragraph Calcpace highlight linking to https://calcpace.app and the gem. `/projects/` stays as a small page with the same links so old inbound links still resolve. `/projects/alljobs/`, `/projects/calcpace/`, `/projects/woche/` (empty pages) are the only URLs allowed to disappear.
- Language filter runs on the home page and on `/tags/`. Paginated `/blog/pageN/` pages show badges only.
- Moonwalk loads Geist fonts from Google Fonts. Kept: it is the theme's typography, one preconnected stylesheet, not a JS loader.

Work happens on branch `moonwalk` (already created, contains the spec). All paths are relative to the repo root `~/Code/jonjo/0jonjo.github.io`. Scratch space: `$SCRATCH=/tmp/claude-1000/-home-jonjo-Code-jonjo/a08e2a6f-6b9a-42dd-9c1a-160a6cdff222/scratchpad`; a Moonwalk checkout at commit `abab9f3` already exists at `$SCRATCH/moonwalk` (if missing: `git clone --depth 1 https://github.com/abhinavs/moonwalk.git $SCRATCH/moonwalk`).

---

## File structure

Created:
- `.github/workflows/pages.yml` — build + deploy + daily cron
- `_data/home.yml` — navbar entries
- `_includes/post_list.html` — post list with `data-lang` + badge (replaces Moonwalk's)
- `_includes/lang_filter.html` — All/EN/PT buttons + inline script
- `_includes/lang_badge.html` — one badge span, used by list and post header
- `_includes/back_link.html` — the back arrow used by inner pages
- `_includes/footer.html` — GitHub, LinkedIn, encoded email, feed
- `_includes/lang_feed.xml` — Atom body shared by the two language feeds
- `en/index.html`, `pt/index.html` — filtered lists
- `en/feed.xml`, `pt/feed.xml` — filtered feeds
- `_sass/custom.scss` — badge + filter styles
- `script/set_lang.rb`, `script/normalize_tags.rb` — one-shot migrations, deleted after use

Copied from Moonwalk: `_layouts/{default,post}.html`, `_includes/{head,horizontal_list,date_and_social_share,post_nav,toc,progress_bar,back_to_top,code_copy,github_alerts,footnotes,toggle_theme_button,toggle_theme_js,custom_head}.html`, `_sass/{moonwalk,list,syntax}.scss`, `assets/css/main.scss`. `_layouts/home.html` is rewritten.

Modified: `_config.yml`, `Gemfile`, `index.md`, `blog/index.html`, `archive.html`, `tags.html`, `_pages/404.md`, `_pages/projects.md`, `README.md`, `.gitignore`, all 61 `_posts/*.md` (front matter only).

Deleted: `_layouts/{about,clean,compress}.html`, `_includes/{analytics,disqus,favicon,mathjax,navigation,social-footer}.html`, `_projects/`, `assets/css/_sass/`, `assets/css/fonts/`, `assets/images/404.png`, `assets/images/profile.png`, `.vscode/`, `_site/`.

Kept: `_includes/youtubePlayer.html` (3 posts use it), favicons (moved), `site.webmanifest` (moved), `LICENSE`.

---

### Task 1: Fix the local build and capture the URL baseline

**Files:**
- Modify: `Gemfile`
- Modify: `_config.yml` (line 7, `url:`)
- Create: `$SCRATCH/baseline_urls.txt`

- [ ] **Step 1: Reproduce the broken build**

Run: `bundle exec jekyll build -d $SCRATCH/old_site 2>&1 | head -3`
Expected: `Could not find sass-embedded-1.69.5 ... (Bundler::GemNotFound)`

- [ ] **Step 2: Rewrite the Gemfile**

```ruby
# frozen_string_literal: true

source "https://rubygems.org"

gem "jekyll", "~> 4.4"

group :jekyll_plugins do
  gem "jekyll-email-protect"
  gem "jekyll-feed"
  gem "jekyll-paginate"
  gem "jekyll-seo-tag"
  gem "jekyll-sitemap"
  gem "jekyll-target-blank"
end

# Ruby 3.4 no longer bundles these as default gems
gem "base64"
gem "bigdecimal"
gem "csv"
gem "logger"

gem "webrick", "~> 1.8"

group :development do
  gem "html-proofer", "~> 5.0"
end
```

- [ ] **Step 3: Set the site URL**

In `_config.yml` replace line 7 `url: #custom url to be used instead of GitHub repository` with:

```yaml
url: https://0jonjo.github.io
```

- [ ] **Step 4: Install and build the OLD site**

Run:
```bash
rm -f Gemfile.lock && bundle install --quiet && bundle exec jekyll build -d $SCRATCH/old_site 2>&1 | tail -3
```
Expected: last line `done in N seconds.` No `URI::InvalidURIError`.

- [ ] **Step 5: Save the baseline URL list**

Run:
```bash
(cd $SCRATCH/old_site && find . -name "*.html" | sort) > $SCRATCH/baseline_urls.txt && wc -l $SCRATCH/baseline_urls.txt && grep -c "/blog/20" $SCRATCH/baseline_urls.txt
```
Expected: about 85 lines; 61 post pages under `/blog/20YY/` (60 if today is before 2026-09-07, the future post is skipped).

- [ ] **Step 6: Commit**

```bash
git add Gemfile Gemfile.lock _config.yml
git commit -m "build: fix local Jekyll build on Ruby 3.4 and set site url"
```

---

### Task 2: Vendor Moonwalk and remove Gradfolio

**Files:**
- Copy from `$SCRATCH/moonwalk`: layouts, includes, sass, `assets/css/main.scss`
- Move: favicons into `assets/images/favicon/`
- Delete: Gradfolio layouts/includes/sass/fonts, `_projects/`, `.vscode/`, `_site/`
- Modify: `_config.yml`, `.gitignore`, `_includes/head.html`, `_layouts/default.html`, `_includes/date_and_social_share.html`

- [ ] **Step 1: Copy Moonwalk files**

```bash
M=$SCRATCH/moonwalk
mkdir -p _sass assets/images/favicon
cp $M/_layouts/default.html $M/_layouts/post.html _layouts/
for f in head horizontal_list date_and_social_share post_nav toc progress_bar back_to_top code_copy github_alerts footnotes toggle_theme_button toggle_theme_js custom_head; do cp $M/_includes/$f.html _includes/; done
cp $M/_sass/moonwalk.scss $M/_sass/list.scss $M/_sass/syntax.scss _sass/
cp $M/assets/css/main.scss assets/css/main.scss
```

- [ ] **Step 2: Delete Gradfolio files, projects collection and stale artifacts**

```bash
git rm -q -r _layouts/about.html _layouts/clean.html _layouts/compress.html \
  _includes/analytics.html _includes/disqus.html _includes/favicon.html \
  _includes/mathjax.html _includes/navigation.html _includes/social-footer.html \
  _projects assets/css/_sass assets/css/fonts assets/images/404.png assets/images/profile.png .vscode
rm -rf _site .jekyll-cache .ruby-lsp
```

- [ ] **Step 3: Move favicons**

```bash
git mv android-chrome-192x192.png android-chrome-512x512.png apple-touch-icon.png favicon-16x16.png favicon-32x32.png site.webmanifest assets/images/favicon/
```
Keep `favicon.ico` at the root (browsers request `/favicon.ico`).

- [ ] **Step 4: Trim `_includes/head.html` to the icons we have**

Replace the block between the two `<!-- Favicon -->` comments with:

```html
  <!-- Favicon -->
  <link rel="apple-touch-icon" sizes="180x180" href="{{ "/assets/images/favicon/apple-touch-icon.png" | relative_url }}">
  <link rel="icon" type="image/png" sizes="32x32" href="{{ "/assets/images/favicon/favicon-32x32.png" | relative_url }}">
  <link rel="icon" type="image/png" sizes="16x16" href="{{ "/assets/images/favicon/favicon-16x16.png" | relative_url }}">
  <link rel="manifest" href="{{ "/assets/images/favicon/site.webmanifest" | relative_url }}">
  <link rel="shortcut icon" href="{{ "/favicon.ico" | relative_url }}">
  <meta name="theme-color" content="#ffffff">
  <!-- Favicon -->
```

Also delete the 3-line `{%- if site.soopr -%} ... {%- endif -%}` dns-prefetch block.

- [ ] **Step 5: Fix `<html lang>` fallback and drop soopr from `_layouts/default.html`**

Line 2 becomes:

```html
<html lang="{{ page.lang | default: site.lang }}" class="html" data-theme="{{ site.theme_config.appearance | default: "auto" }}">
```

Delete both `{%- if site.soopr -%} ... {%- endif -%}` blocks (one inside `.credits`, one before `</body>`) and the `{%- if site.theme_config.show_link_previews ... -%}` block.

- [ ] **Step 6: Remove the soopr button from `_includes/date_and_social_share.html`**

Delete the final `<div class="soopr-btn" ...></div>` (3 lines).

- [ ] **Step 7: Rewrite `_config.yml`**

```yaml
title: João Gilberto Saraiva
author: João Gilberto Saraiva
description: software engineer | professor | writer
url: https://0jonjo.github.io
baseurl: ""
lang: pt
email: jgilbertons@gmail.com
linkedin: 0jonjo
github: 0jonjo

permalink: /blog/:year/:title/
paginate: 4
paginate_path: /blog/page:num/

theme_config:
  appearance: "light"
  appearance_toggle: true
  date_format: "%Y-%m-%d"
  show_description: true
  show_navbar: true
  show_footer: true
  show_copyright: true
  show_reading_time: true
  show_tags: true
  show_code_copy: true
  show_progress_bar: true
  show_back_to_top: true
  show_post_nav: true
  show_toc: true
  show_footnote_tooltips: true
  show_link_previews: false

markdown: kramdown
highlighter: rouge

sass:
  style: compressed

include:
  - _pages

exclude:
  - README.md
  - LICENSE
  - docs
  - script
  - Gemfile
  - Gemfile.lock
  - vendor

plugins:
  - jekyll-seo-tag
  - jekyll-feed
  - jekyll-sitemap
  - jekyll-paginate
  - jekyll-email-protect
  - jekyll-target-blank
```

- [ ] **Step 8: `.gitignore`**

```
_site/
.sass-cache/
.jekyll-cache/
.jekyll-metadata
.ruby-lsp/
vendor/
```

- [ ] **Step 9: Build. Pages still on old layouts will error, that is Task 3's job. Confirm the theme compiles.**

Run: `bundle exec jekyll build -d $SCRATCH/new_site 2>&1 | grep -iE "error|warn|done" | head`
Expected: `done in N seconds.` or errors only about `about`/`clean` layouts not found. No Sass errors.

- [ ] **Step 10: Commit**

```bash
git add -A
git commit -m "theme: vendor Moonwalk layouts, includes and styles; drop Gradfolio and projects collection"
```

---

### Task 3: Rebuild the site pages on Moonwalk

**Files:**
- Create: `_data/home.yml`, `_includes/back_link.html`, `_includes/footer.html`, `_includes/post_list.html`
- Modify: `index.md`, `blog/index.html`, `archive.html`, `tags.html`, `_pages/404.md`, `_pages/projects.md`, `_layouts/home.html`

- [ ] **Step 1: `_data/home.yml`**

```yaml
navbar_entries:
  - title: home
    url: /
  - title: posts
    url: /blog/
  - title: tags
    url: /tags/
  - title: archive
    url: /archive/
```

- [ ] **Step 2: `_includes/back_link.html`**

```html
<a href="{{ "/" | relative_url }}" class="back-link" aria-label="Back to home">
  <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="15 18 9 12 15 6"></polyline></svg>
</a>
```

- [ ] **Step 3: `_includes/footer.html`**

```html
<footer>
  <div class="dashed"></div>
  <ul class="horizontal-list">
    <li><a href="https://github.com/{{ site.github }}" target="_blank" rel="noopener noreferrer">github</a>&nbsp;&nbsp;</li>
    <li><a href="https://www.linkedin.com/in/{{ site.linkedin }}" target="_blank" rel="noopener noreferrer">linkedin</a>&nbsp;&nbsp;</li>
    <li><a href="mailto:{{ site.email | encode_email }}">email</a>&nbsp;&nbsp;</li>
    <li><a href="{{ "/feed.xml" | relative_url }}">feed</a>&nbsp;&nbsp;</li>
  </ul>
</footer>
```

- [ ] **Step 4: `_layouts/home.html`** (rewritten)

```html
---
layout: default
---

<header>
{% if site.theme_config.show_navbar == true %}
  {% include horizontal_list.html collection=site.data.home.navbar_entries %}
  <div class="dashed"></div>
{% endif %}
  <h1>{{ site.title }}</h1>
  {% if site.theme_config.show_description == true %}
    <p>{{ site.description }}</p>
  {% endif %}
</header>

{{ content }}

{% if site.theme_config.show_footer == true %}
  {% include footer.html %}
{% endif %}
```

- [ ] **Step 5: `_includes/post_list.html`** (accepts `include.posts`, defaults to all posts; badge arrives in Task 6)

```html
{% assign posts = include.posts | default: site.posts %}
<ul class="post-list" data-post-list>
  {% for post in posts %}
    <li class="post-list-item" data-lang="{{ post.lang | default: site.lang }}">
      <span class="home-date">{{ post.date | date: site.theme_config.date_format }}&raquo;</span>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
```

- [ ] **Step 6: `index.md`**

```markdown
---
layout: home
permalink: /
---

I build [Calcpace](https://calcpace.app), a running and cycling pace calculator, and maintain the [calcpace gem](https://github.com/0jonjo/calcpace) it grew from.

## Posts
{% include post_list.html %}
```

- [ ] **Step 7: `blog/index.html`** (paginated, URL-compatible)

```html
---
layout: default
title: Posts
---

{% include back_link.html %}

<header><h1>Posts</h1></header>

{% include post_list.html posts=paginator.posts %}

{% if paginator.total_pages > 1 %}
<nav class="pagination horizontal-list">
  {% if paginator.previous_page %}<a href="{{ paginator.previous_page_path | relative_url }}">&laquo; prev</a>{% endif %}
  <span>page {{ paginator.page }} of {{ paginator.total_pages }}</span>
  {% if paginator.next_page %}<a href="{{ paginator.next_page_path | relative_url }}">next &raquo;</a>{% endif %}
</nav>
{% endif %}
```

- [ ] **Step 8: `archive.html`**

```html
---
layout: default
permalink: /archive/
title: Archive
---

{% include back_link.html %}

<header><h1>Archive</h1></header>

{% assign by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" %}
{% for year in by_year %}
  <h2 id="y{{ year.name }}">{{ year.name }}</h2>
  {% include post_list.html posts=year.items %}
{% endfor %}
```

- [ ] **Step 9: `tags.html`**

Copy `$SCRATCH/moonwalk/tags.html` over `tags.html`, then:
- front matter `permalink: /tags` → `permalink: /tags/`
- replace the Moonwalk back-link `<a ...>...</a>` block (the 5 lines after the front matter) with `{% include back_link.html %}`
- inside each `tag-section`, replace the `<ul> ... </ul>` block with `{% include post_list.html posts=tag[1] %}`

- [ ] **Step 10: `_pages/projects.md`** (URL kept, content is the Calcpace highlight)

```markdown
---
title: Projects
layout: default
permalink: /projects/
---

{% include back_link.html %}

<header><h1>Projects</h1></header>

<ul>
  <li><a href="https://calcpace.app" target="_blank" rel="noopener noreferrer">Calcpace</a> — pace, distance and time calculator for runners and cyclists.</li>
  <li><a href="https://github.com/0jonjo/calcpace" target="_blank" rel="noopener noreferrer">calcpace gem</a> — the Ruby library behind it: pace math and unit conversion.</li>
</ul>
```

- [ ] **Step 11: `_pages/404.md`**

Copy `$SCRATCH/moonwalk/404.html` content into `_pages/404.md` (front matter already has `permalink: /404.html`). Change the lede to `The page you were looking for is not here. Maybe it moved, maybe it never existed.`

- [ ] **Step 12: Build and diff URLs**

Run:
```bash
bundle exec jekyll build -d $SCRATCH/new_site 2>&1 | tail -1
(cd $SCRATCH/new_site && find . -name "*.html" | sort) > $SCRATCH/new_urls.txt
comm -23 $SCRATCH/baseline_urls.txt $SCRATCH/new_urls.txt
```
Expected: `done in N seconds.`; `comm` prints exactly three lines: `./projects/alljobs/index.html`, `./projects/calcpace/index.html`, `./projects/woche/index.html`. Anything else missing is a regression (if `./blog/page2/index.html` is missing, pagination broke: check `blog/index.html` exists and `paginate_path` in config).

- [ ] **Step 13: Quick HTTP check**

Run: `bundle exec jekyll serve -d $SCRATCH/new_site --port 4001 --detach >/dev/null && curl -s localhost:4001/ | grep -c "post-list-item"; curl -s -o /dev/null -w "%{http_code}\n" localhost:4001/blog/2026/polished-ruby-programming/; pkill -f "jekyll serve"`
Expected: `60` or `61` (see Task 1 note on the future post), then `200`.

- [ ] **Step 14: Commit**

```bash
git add -A
git commit -m "pages: rebuild home, posts, archive, tags, projects and 404 on Moonwalk"
```

---

### Task 4: Add `lang` to every post

**Files:**
- Create: `script/set_lang.rb` (deleted at end of task)
- Modify: `_posts/*.md` (front matter only)

- [ ] **Step 1: Write the script**

```ruby
# frozen_string_literal: true

# One-shot: derive `lang` from the `english` tag, then drop that tag.
Dir["_posts/*.md"].sort.each do |path|
  text = File.read(path)
  next if text.match?(/^lang: /)

  tags_line = text[/^tags:.*$/]
  tags = tags_line.to_s.sub(/^tags:\s*/, "").split
  lang = tags.include?("english") ? "en" : "pt"
  tags.delete("english")

  text = text.sub(/^tags:.*$/, "tags: #{tags.join(' ')}") if tags_line
  text = text.sub(/^layout: post\n/, "layout: post\nlang: #{lang}\n")
  File.write(path, text)
  puts "#{lang}  #{File.basename(path)}"
end
```

- [ ] **Step 2: Run it**

Run: `mkdir -p script && ruby script/set_lang.rb | cut -c1-2 | sort | uniq -c`
Expected: `25 en` and `36 pt`.

- [ ] **Step 3: Verify**

Run: `grep -l "^lang: en" _posts/*.md | wc -l; grep -l "^lang: pt" _posts/*.md | wc -l; grep -l "^tags:.*english" _posts/*.md | wc -l; git diff --stat | tail -1`
Expected: `25`, `36`, `0`, `61 files changed`.

- [ ] **Step 4: Remove the script and commit**

```bash
rm script/set_lang.rb && rmdir script
git add -A
git commit -m "posts: add lang front matter (en/pt), drop english tag"
```

---

### Task 5: Normalize tags

**Files:**
- Create: `script/normalize_tags.rb` (deleted at end of task)
- Modify: `_posts/*.md` (tags line only)

Mapping (Portuguese or misspelled → English, lowercase, one word):

| from | to |
|---|---|
| programacao, progamacao | programming |
| refatoracao | refactoring |
| projeto | project |
| comunidade, comunidades | community |
| literatura | literature |
| livro | books |
| linguas | languages |
| alemao | german |
| ensino | teaching |
| psicologia | psychology |
| carreira | career |
| queueing | queues |
| polimorfismo | polymorphism |
| rubyonrails | rails |

Unchanged: programming, learning, ruby, ai, history, python, java, gem, queues, campuscode, duolingo.

- [ ] **Step 1: Write the script**

```ruby
# frozen_string_literal: true

MAP = {
  "programacao" => "programming", "progamacao" => "programming",
  "refatoracao" => "refactoring", "projeto" => "project",
  "comunidade" => "community", "comunidades" => "community",
  "literatura" => "literature", "livro" => "books", "linguas" => "languages",
  "alemao" => "german", "ensino" => "teaching", "psicologia" => "psychology",
  "carreira" => "career", "queueing" => "queues",
  "polimorfismo" => "polymorphism", "rubyonrails" => "rails"
}.freeze

Dir["_posts/*.md"].sort.each do |path|
  text = File.read(path)
  tags_line = text[/^tags:.*$/] or next
  tags = tags_line.sub(/^tags:\s*/, "").split.map { |t| MAP.fetch(t, t) }.uniq
  new_line = "tags: #{tags.join(' ')}"
  next if new_line == tags_line

  File.write(path, text.sub(tags_line, new_line))
  puts "#{File.basename(path)}: #{tags_line} -> #{new_line}"
end
```

- [ ] **Step 2: Run and verify**

Run: `mkdir -p script && ruby script/normalize_tags.rb | wc -l; grep -h "^tags:" _posts/*.md | sed 's/tags: *//' | tr ' ' '\n' | grep -v '^$' | sort -u | tr '\n' ' '`
Expected: about 14 changed files; the printed tag list contains none of the "from" column and no `english`.

- [ ] **Step 3: Remove the script and commit**

```bash
rm script/normalize_tags.rb && rmdir script
git add -A
git commit -m "posts: normalize tags to lowercase English"
```

---

### Task 6: Language UI: badge, filter, /en/ and /pt/ pages, per-language feeds

**Files:**
- Create: `_includes/lang_badge.html`, `_includes/lang_filter.html`, `_includes/lang_feed.xml`, `_sass/custom.scss`, `en/index.html`, `pt/index.html`, `en/feed.xml`, `pt/feed.xml`
- Modify: `_includes/post_list.html`, `_includes/date_and_social_share.html`, `assets/css/main.scss`, `index.md`, `tags.html`, `_data/home.yml`

- [ ] **Step 1: `_includes/lang_badge.html`**

```html
{% assign badge_lang = include.lang | default: site.lang %}
<span class="lang-badge" data-lang-badge="{{ badge_lang }}" title="{% if badge_lang == 'en' %}Written in English{% else %}Escrito em português{% endif %}">{{ badge_lang | upcase }}</span>
```

- [ ] **Step 2: Badge in `_includes/post_list.html`**

Inside the `<li>`, after the `<a>...</a>` line, add:

```html
      {% include lang_badge.html lang=post.lang %}
```

- [ ] **Step 3: Badge in the post header**

In `_includes/date_and_social_share.html`, inside `<div class="post-meta-line">`, right after the `{% endif %}` closing the reading-time block, add:

```html
    <span class="post-meta-sep">·</span>
    {% include lang_badge.html lang=page.lang %}
```

- [ ] **Step 4: `_includes/lang_filter.html`**

```html
<div class="lang-filter" data-lang-filter role="group" aria-label="Filter posts by language">
  <button type="button" data-lang="all" class="is-active">All</button>
  <button type="button" data-lang="en">EN</button>
  <button type="button" data-lang="pt">PT</button>
</div>
<script>
(function () {
  var KEY = "postLang";
  var filter = document.querySelector("[data-lang-filter]");
  if (!filter) return;

  function apply(lang) {
    document.querySelectorAll("[data-post-list] [data-lang]").forEach(function (li) {
      li.hidden = lang !== "all" && li.dataset.lang !== lang;
    });
    document.querySelectorAll(".tag-section").forEach(function (section) {
      section.hidden = !section.querySelector("[data-lang]:not([hidden])");
    });
    filter.querySelectorAll("button").forEach(function (button) {
      button.classList.toggle("is-active", button.dataset.lang === lang);
    });
  }

  var saved = "all";
  try { saved = localStorage.getItem(KEY) || "all"; } catch (e) {}
  apply(saved);

  filter.addEventListener("click", function (event) {
    var button = event.target.closest("button[data-lang]");
    if (!button) return;
    try { localStorage.setItem(KEY, button.dataset.lang); } catch (e) {}
    apply(button.dataset.lang);
  });
})();
</script>
```

- [ ] **Step 5: `_sass/custom.scss`**

```scss
.lang-badge {
  font-family: var(--font-mono);
  font-size: 0.65em;
  padding: 0.1em 0.45em;
  margin-left: 0.5em;
  border-radius: var(--radius-sm);
  border: 1px solid var(--border-subtle);
  background-color: var(--bg-subtle);
  color: var(--text-secondary);
  vertical-align: middle;
}

.lang-filter {
  display: flex;
  gap: 0.5em;
  margin: 0.5em 0 1em;

  button {
    font-family: var(--font-mono);
    font-size: 0.8em;
    padding: 0.25em 0.9em;
    border-radius: 1em;
    border: 1px solid var(--border);
    background-color: var(--bg-subtle);
    color: var(--text-secondary);
    cursor: pointer;

    &.is-active {
      border-color: var(--links);
      color: var(--links);
      font-weight: 600;
    }
  }
}

.pagination {
  margin-top: 2em;
  gap: 1em;
}

.post-list-item[hidden],
.tag-section[hidden] { display: none; }
```

- [ ] **Step 6: Import it in `assets/css/main.scss`**

```scss
---
---

@use "moonwalk";
@use "list";
@use "syntax";
@use "custom";
```

- [ ] **Step 7: Filter on the home page and tags page**

`index.md` becomes:

```markdown
---
layout: home
permalink: /
---

I build [Calcpace](https://calcpace.app), a running and cycling pace calculator, and maintain the [calcpace gem](https://github.com/0jonjo/calcpace) it grew from.

## Posts
{% include lang_filter.html %}
{% include post_list.html %}
```

In `tags.html`, right after `<h1>Tags</h1>` add `{% include lang_filter.html %}`.

- [ ] **Step 8: `en/index.html`**

```html
---
layout: default
title: Posts in English
permalink: /en/
lang: en
---

{% include back_link.html %}

<header><h1>Posts in English</h1></header>

{% assign posts = site.posts | where: "lang", "en" %}
{% include post_list.html posts=posts %}

<p><a href="{{ "/en/feed.xml" | relative_url }}">Atom feed for English posts</a></p>
```

- [ ] **Step 9: `pt/index.html`**

```html
---
layout: default
title: Posts em português
permalink: /pt/
lang: pt
---

{% include back_link.html %}

<header><h1>Posts em português</h1></header>

{% assign posts = site.posts | where: "lang", "pt" %}
{% include post_list.html posts=posts %}

<p><a href="{{ "/pt/feed.xml" | relative_url }}">Feed Atom dos posts em português</a></p>
```

- [ ] **Step 10: `_includes/lang_feed.xml`**

```xml
<?xml version="1.0" encoding="utf-8"?>
<feed xmlns="http://www.w3.org/2005/Atom" xml:lang="{{ include.lang }}">
  <title>{{ site.title | xml_escape }} ({{ include.lang | upcase }})</title>
  <link href="{{ page.url | absolute_url }}" rel="self" type="application/atom+xml" />
  <link href="{{ include.lang | prepend: '/' | append: '/' | absolute_url }}" rel="alternate" type="text/html" />
  <updated>{{ site.time | date_to_xmlschema }}</updated>
  <id>{{ page.url | absolute_url }}</id>
  <author><name>{{ site.author | xml_escape }}</name></author>
  {% assign posts = site.posts | where: "lang", include.lang %}
  {% for post in posts limit: 20 %}
  <entry>
    <title>{{ post.title | xml_escape }}</title>
    <link href="{{ post.url | absolute_url }}" rel="alternate" type="text/html" />
    <id>{{ post.url | absolute_url }}</id>
    <published>{{ post.date | date_to_xmlschema }}</published>
    <updated>{{ post.date | date_to_xmlschema }}</updated>
    <summary>{{ post.description | xml_escape }}</summary>
    <content type="html">{{ post.content | xml_escape }}</content>
  </entry>
  {% endfor %}
</feed>
```

- [ ] **Step 11: `en/feed.xml` and `pt/feed.xml`**

`en/feed.xml`:
```
---
layout: null
permalink: /en/feed.xml
---
{% include lang_feed.xml lang="en" %}
```

`pt/feed.xml`:
```
---
layout: null
permalink: /pt/feed.xml
---
{% include lang_feed.xml lang="pt" %}
```

- [ ] **Step 12: EN / PT in the navbar**

Append to `navbar_entries` in `_data/home.yml`:

```yaml
  - title: en
    url: /en/
  - title: pt
    url: /pt/
```

- [ ] **Step 13: Build and verify**

Run:
```bash
bundle exec jekyll build -d $SCRATCH/new_site 2>&1 | tail -1
grep -c 'data-lang="en"' $SCRATCH/new_site/index.html
grep -c 'data-lang="pt"' $SCRATCH/new_site/index.html
grep -c "<entry>" $SCRATCH/new_site/en/feed.xml $SCRATCH/new_site/pt/feed.xml
grep -o '<html lang="[a-z]*"' $SCRATCH/new_site/blog/2026/polished-ruby-programming/index.html $SCRATCH/new_site/blog/2021/calcpace/index.html
grep -c "lang-badge" $SCRATCH/new_site/blog/2026/polished-ruby-programming/index.html
ruby -rrexml/document -e 'REXML::Document.new(File.read(ARGV[0])); puts "xml ok"' $SCRATCH/new_site/en/feed.xml
```
Expected: `25` (24 before 2026-09-07), `36`, `en/feed.xml:20` and `pt/feed.xml:20`, `lang="en"` for the 2026 post and `lang="pt"` for the 2021 post, `1`, `xml ok`.

- [ ] **Step 14: Commit**

```bash
git add -A
git commit -m "i18n: language badge, All/EN/PT filter, /en and /pt lists and feeds"
```

---

### Task 7: Link check with html-proofer

- [ ] **Step 1: Run html-proofer on the built site**

Run:
```bash
bundle exec jekyll build -d $SCRATCH/new_site 2>&1 | tail -1
bundle exec htmlproofer $SCRATCH/new_site --disable-external --no-enforce-https --ignore-urls "/^javascript:/" --allow-missing-href 2>&1 | tail -15
```
Expected: `HTML-Proofer finished successfully.` If it reports broken internal links or missing images, fix the source (likely: a post linking to a deleted `/assets/images/...` file, or a broken `#anchor`) and rerun. Rewrite links to the new path, do not delete them.

- [ ] **Step 2: Commit any fixes**

```bash
git add -A
git commit -m "fix: internal links flagged by html-proofer"
```
(Skip if nothing changed.)

---

### Task 8: GitHub Actions deploy with daily cron

**Files:**
- Create: `.github/workflows/pages.yml`
- Modify: `README.md`; add `.tool-versions` to git

- [ ] **Step 1: Write the workflow**

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  schedule:
    - cron: "0 9 * * *" # daily 09:00 UTC so future-dated posts publish on their date
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: "3.4"
          bundler-cache: true

      - name: Build site
        run: bundle exec jekyll build --trace
        env:
          JEKYLL_ENV: production

      - uses: actions/upload-pages-artifact@v3
        with:
          path: _site

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

- [ ] **Step 2: Validate the YAML**

Run: `ruby -ryaml -e 'y = YAML.load_file(".github/workflows/pages.yml"); puts y["on"].keys.join(","), y["jobs"].keys.join(",")'`
Expected: `push,schedule,workflow_dispatch` and `build,deploy`. (Ruby parses the bare `on:` key as the boolean `true`; if the first line prints nothing, use `y[true]` and treat it as fine.)

- [ ] **Step 3: README**

```markdown
# 0jonjo.github.io

Personal blog. Jekyll 4 with a vendored copy of the [Moonwalk](https://github.com/abhinavs/moonwalk) theme.

## Local

    bundle install
    bundle exec jekyll serve

## Deploy

Pushes to `main` and a daily cron (09:00 UTC) run `.github/workflows/pages.yml`, which builds with Jekyll 4 and deploys to GitHub Pages. Posts with a future date go live on the first run after their date.

## Writing

Posts live in `_posts/YYYY-MM-DD-slug.md` with `lang: en` or `lang: pt` in the front matter. Tags are lowercase English words.
```

- [ ] **Step 4: Commit**

```bash
git add .github/workflows/pages.yml README.md .tool-versions
git commit -m "ci: build and deploy with GitHub Actions, daily cron for scheduled posts"
```

---

### Task 9: Final verification, PR, Pages source switch

- [ ] **Step 1: Clean build + URL diff**

Run:
```bash
rm -rf $SCRATCH/new_site && bundle exec jekyll build -d $SCRATCH/new_site 2>&1 | grep -iE "warn|error|done"
(cd $SCRATCH/new_site && find . -name "*.html" | sort) > $SCRATCH/new_urls.txt
echo "missing:"; comm -23 $SCRATCH/baseline_urls.txt $SCRATCH/new_urls.txt
echo "new:"; comm -13 $SCRATCH/baseline_urls.txt $SCRATCH/new_urls.txt
```
Expected: `done`, no warnings. `missing:` is exactly the three `./projects/<name>/index.html` lines. `new:` is `./en/index.html`, `./pt/index.html` (plus the 2026-09-07 post if built on or after that date).

- [ ] **Step 2: Confirm dead weight is gone**

Run: `grep -rlE "fontawesome|mathjax|ga-lite|disqus|soopr" $SCRATCH/new_site | wc -l; ls $SCRATCH/new_site/feed.xml $SCRATCH/new_site/sitemap.xml`
Expected: `0`, then both files listed.

- [ ] **Step 3: Visual check in Chrome**

Run: `bundle exec jekyll serve -d $SCRATCH/new_site --port 4001 --detach >/dev/null`
Open and screenshot with the Chrome tools: `http://localhost:4001/`, `/blog/2026/polished-ruby-programming/` (en), `/blog/2021/calcpace/` (pt), `/tags/`, `/en/`, `/projects/`, `/blog/page2/`, `/nope` (404). On the home page click `PT`, reload, confirm only PT posts stay visible and the `PT` button is highlighted. Then `pkill -f "jekyll serve"`.
Fix anything visibly broken, commit as `fix: ...`.

- [ ] **Step 4: Push and open the PR**

```bash
git push -u origin moonwalk
gh pr create --title "Migrate to Moonwalk theme, GitHub Actions deploy, pt/en separation" --body-file docs/superpowers/specs/2026-09-06-moonwalk-migration-design.md
```

- [ ] **Step 5: Switch Pages to Actions (confirm with the user first)**

```bash
gh api -X PUT repos/0jonjo/0jonjo.github.io/pages -f build_type=workflow
gh api repos/0jonjo/0jonjo.github.io/pages --jq .build_type
```
Expected: `workflow`. Do this right before merging: once switched, the legacy build stops and the site only updates when the workflow on `main` runs.

- [ ] **Step 6: Merge, watch the run, smoke the live site**

```bash
gh pr merge --squash --delete-branch
gh run watch $(gh run list --workflow pages.yml --limit 1 --json databaseId --jq '.[0].databaseId')
curl -s -o /dev/null -w "%{http_code}\n" https://0jonjo.github.io/
curl -s -o /dev/null -w "%{http_code}\n" https://0jonjo.github.io/blog/2026/polished-ruby-programming/
curl -s https://0jonjo.github.io/en/feed.xml | grep -c "<entry>"
```
Expected: run `completed success`, `200`, `200`, `20`.

- [ ] **Step 7: Confirm the scheduled post**

On or after 2026-09-07: `gh workflow run pages.yml && sleep 120 && curl -s -o /dev/null -w "%{http_code}\n" https://0jonjo.github.io/blog/2026/translating-documents-ruby-llm/`
Expected: `200`. Then delete `~/Code/jonjo/blog_posts_agendados.md` (its pending item is resolved).

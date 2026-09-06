# 0jonjo.github.io

Personal blog. Jekyll 4 with a vendored copy of the [Moonwalk](https://github.com/abhinavs/moonwalk) theme.

## Local

    bundle install
    bundle exec jekyll serve

## Deploy

Pushes to `main` and a daily cron (09:00 UTC) run `.github/workflows/pages.yml`, which builds with Jekyll 4 and deploys to GitHub Pages. Posts with a future date go live on the first run after their date.

## Writing

Posts live in `_posts/YYYY-MM-DD-slug.md`. Front matter:

    layout: post
    lang: en          # or pt
    locale: en_US     # or pt_BR (used for og:locale)
    title: "..."
    description: "..."
    tags: programming ruby
    image: https://...

Tags are lowercase English words. The tags page only lists tags with more than 3 posts.

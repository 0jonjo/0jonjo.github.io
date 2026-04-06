---
layout: post
title: "Calcpace: open beta, maps, bot protection and a growing gem"
description: "From closed beta to open registration: new gem modules, GPS tracking with maps, bot protection, and the infrastructure decisions behind each feature."
tags: programming ruby english
categories: misc
youtubeId:
image: https://github.com/user-attachments/assets/da5c62bd-2290-4118-99a4-ddf27140c870
---

Calcpace.app is now open to anyone — no invite code needed. This post covers what changed since the closed beta: new gem modules, GPS tracking with maps, bot protection, the infrastructure decisions behind each feature, and a few lessons learned the hard way.

The `calcpace` gem is a pure Ruby library — no I/O, no Rails, no external dependencies. Every formula lives in its own module, included into the main `Calcpace` class. The site consumes it as a regular gem dependency; when a new module ships, the site updates the `Gemfile` and builds on top of it. Since v1.8, three modules were added.

**CameronPredictor** — an exponential race prediction formula that runs alongside the existing Riegel predictor (`T2 = T1 × (D2/D1)^1.06`). Cameron tends to be more conservative than Riegel for shorter base distances, which matters when you're predicting a marathon from a 5K time. Having both lets you compare and pick the more realistic estimate for your training context.

**TrackCalculator** — GPS math: Haversine distance between coordinate pairs, cumulative elevation gain, and per-km splits from an array of raw trackpoints. Pure calculation — no map rendering, no file parsing. The module receives coordinates and returns numbers; the site handles everything else.

**Vo2maxEstimator** — Daniels & Gilbert formula. Given a race time and distance, returns an estimated VO2max and a fitness classification (from "Poor" to "Elite"). The gem API convention applies here: inputs use symbols (`:km`, `:mi`), outputs are always strings — never symbols.

The GPS flow on the site: when a user uploads a GPX file, Active Storage saves it to Cloudflare R2 and enqueues `GpxParseJob` to Sidekiq. The job runs `GpxParser`, which extracts trackpoints (lat, lon, elevation, time), then calls `TrackCalculator` from the gem to compute distance, elevation gain, and splits. Trackpoints are persisted and the activity is updated with the computed stats. The map renders with Leaflet.js and OpenStreetMap tiles — no API key, no vendor lock-in, no usage limits.

Moving from invite-only to open registration meant adding real bot protection. The approach: Cloudflare Turnstile on the registration form, Rack::Attack for rate limiting at the Rails level. Turnstile was straightforward to integrate, with two gotchas that cost more time than expected. The widget script URL uses `/v0/`, not `/v1/` — the docs aren't always consistent about this. And the form param is `cf-turnstile-response` (hyphens), not `cf_turnstile_response` (underscores) — Rails' `params` hash keeps hyphens, so the usual convention doesn't apply. There's also a pending issue: when the registration form reloads after a validation error, Turbo replaces the DOM but doesn't re-initialize the widget, so the next submission fails silently. The fix is re-initializing after a Turbo render — still pending.

For file storage, Active Storage with Cloudflare R2 as the backend handles avatars, activity photos, and GPX files. S3-compatible, free egress within Cloudflare's network, no CDN configuration needed.

Rails 8 ships `generates_token_for` as a first-class model API for signed, expiring tokens. The email verification flow uses it: on registration a token is generated with 48h expiry, the verification email includes a signed URL, and unverified accounts are blocked from logging in with a resend option shown. The verification email is HTML now — styled with a button and a copyable fallback link. Before this it was plain text, which looked unfinished for an open product. Transactional emails go through Resend API; the password reset flow was already using it and verification plugged into the same setup.

Every user gets a public profile at `calcpace.app/:username` — avatar, bio, city, country, and activity feed. Each activity has an individual `public` boolean: toggle it on the activity form and it appears or disappears from the public feed immediately. Activities can also have a photo attached. Your profile page only shows what you choose to share.

The clearest architectural decision in this project: the gem does pure calculation, the site does I/O. No file parsing in the gem, no HTTP, no database. This makes the gem testable in isolation (Minitest, no Rails required) and keeps the site thin — controllers call gem methods and persist results, nothing more. The delivery sequence is always: implement module in gem → write tests → publish new version → update `Gemfile` in site → build the UI on top. GitHub Actions publishes to RubyGems automatically on version bump in `lib/calcpace/version.rb`.

What's next: internationalization first — the app is English-only for now, but the i18n infrastructure is already in place. Adding PT-BR, Spanish, German, and French means auditing hardcoded strings, migrating `GpxParser` error messages to i18n keys, and translating the emails. Then content pages: pace conversion tables, race equivalent tables, Boston qualifying times, world records. Mile splits in the activity view are also coming — the gem already supports it via `track_splits(points, 1.609)`, the UI just needs a toggle. Strava and Garmin direct connection is planned — GPX import already works for the manual export flow, the next step is OAuth2 for automatic sync.

- [calcpace.app](https://calcpace.app)
- [calcpace gem on RubyGems](https://rubygems.org/gems/calcpace)
- [calcpace_web on GitHub](https://github.com/0jonjo/calcpace_web)

<img width="1024" height="683" alt="image" src="https://github.com/user-attachments/assets/da5c62bd-2290-4118-99a4-ddf27140c870" />

>Image: ["BP Running Track @ Glasgow Airport"](https://openverse.org/image/c512f324-473b-4a6c-b769-0a993b9aa445) by JCDecaux Creative Solutions. [Open Verse, Creative Commons BY-NC-ND 2.0](https://creativecommons.org/licenses/by-nc-nd/2.0/)

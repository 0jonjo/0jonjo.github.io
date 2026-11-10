---
layout: post
lang: en
locale: en_US
title: "Calcpace passed 50 installs on Google Play"
description: "What shipped in the Calcpace Android app since launch, how one Rails 8 and Hotwire codebase runs inside it, and what comes next on iPhone."
tags: programming ruby
categories: misc
youtubeId:
image:
---

[Calcpace](https://calcpace.app) passed 50 installs on [Google Play](https://play.google.com/store/apps/details?id=app.calcpace.twa), from 7 countries and regions: Brazil, Spain, Cambodia, Germany, Portugal, Venezuela and Macau.

## What shipped since launch

- Run recording with the phone's GPS, in a native Kotlin screen.
- Health Connect both ways: workouts from your watch arrive with their route, and runs recorded in Calcpace go back to Health Connect.
- Importing your whole Strava history from the export ZIP.

## How it is built

Apart from that recording screen, the app is still the calcpace.app site: one Rails 8 and Hotwire codebase, inside a [Hotwire Native](https://native.hotwired.dev) shell on Android.

All the math is the [calcpace gem](https://rubygems.org/gems/calcpace). The UI is [jet_ui](https://github.com/jetrockets/jet_ui), which we're building at [JetRockets](https://jetrockets.com).

## Next up: iPhone

Next is the iPhone app, with the same Hotwire Native approach.

<img width="800" height="999" alt="image" src="https://github.com/user-attachments/assets/713abc88-f2c1-4d0d-8d67-41f4990e493e" />

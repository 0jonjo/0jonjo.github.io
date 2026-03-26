---
layout: post
title: "Calcpace Web: the calculator now in the browser"
description: "The calcpace gem just hit 7,000 downloads, so I ran a 5k and built calcpace.app to bring those calculations to everyone, no Ruby required."
tags: programming ruby english
categories: misc
youtubeId:
image:
---

The `calcpace` gem just hit 7,000 downloads, so I celebrated the best way I know how: I ran a 5k and finally built [calcpace.app](https://calcpace.app) to bring those calculations to everyone, no Ruby console required.


The core logic for this project has actually been brewing in my head for about five years. Combining my daily routine as a runner with the logic I apply professionally when building delivery and routing systems, it just made perfect sense to model pace, distance, and time mathematically. I eventually sat down and started writing the core Ruby code four years ago, which evolved into the open-source gem.

When the gem crossed the 7k mark, I realized it was time to make it accessible to non-programmers.


The app provides a suite of free running tools: pace, speed, finish time, split breakdowns, and a race predictor using both the Riegel and Cameron formulas. The converter covers 30 combinations across distance, speed, and pace—km, mi, meters, yards, feet, knots—all easily switchable between metric and imperial systems.

I didn't want it to be just another "enter your numbers" calculator. There are dedicated guide pages explaining the math behind each formula. For instance, you can read about why the Cameron predictor tends to be more conservative than Riegel for shorter base distances, and what that actually means for your training blocks.


From planning to writing the code and setting up the infrastructure, the web version took me about four days. I used this as an opportunity to build with a modern, pragmatic stack:

* **Ruby on Rails 8:** The foundation of the app. It provides a solid structure and allows me to practice "dogfooding" by consuming my own gem in a production environment. The gem does all the heavy lifting for the calculations, keeping the Rails controllers incredibly clean.
* **The Basecamp Way:** I treated this project as a playground to strictly follow 37signals' design principles. That means pushing the logic down to rich models, keeping controllers thin, and leveraging `Current` attributes to handle global state cleanly.
* **Tailwind CSS:** To keep my focus on the backend and infrastructure, I used Tailwind to rapidly build a clean, responsive interface without writing custom CSS files.
* **Testing & CI/CD:** Since this serves as a live environment for the gem, reliability is key. The app is fully tested with Minitest, and GitHub Actions runs the CI/CD pipeline on every push to `main`.
* **Kamal & VPS:** I wanted to step away from traditional, expensive PaaS solutions. I deployed the application using Kamal, which packages the Rails app into Docker containers and handles zero-downtime deployments directly to a DigitalOcean Virtual Private Server (VPS).

### Take it for a spin

While the personal activity tracker (run/ride logging) is technically in closed testing, I built a sandbox and share with some friends so they can explore the UI and see the gem working in a real database environment.

To keep things tidy, a `GuestResetJob` runs via `sidekiq-cron` to wipe the guest activities and recreate the sample profile every Monday at 3 AM.

### What's Next

The engine is running smoothly, but there's more to come. I am currently working on adding VO2max, VDOT, and training zone calculations—these will be introduced as new modules in the gem and will get their own guide pages on the site.

Links:
- [calcpace.app](https://calcpace.app)
- [calcpace gem](https://rubygems.org/gems/calcpace)
- [GitHub repo](https://github.com/0jonjo/calcpace_web)

Image: "Word Search" by Morgan Frederick. [Open Verse, Creative Commons 2.0](https://openverse.org/image/79fc1531-5557-43e8-8314-0ef15b6c36b9)

---
layout: post
title: "From Backend to MLOps: A Runner’s Road to Business Rules, Deployments and Monitoring"
description: "The intersection of Backend Development and MLOps, a deep dive into applying DevOps principles, Design Thinking, and deployment strategies to machine learning systems using the Calcpace app."
tags: programming learning ai english ruby
categories: misc
youtubeId:
image: https://github.com/user-attachments/assets/75629b8e-47f8-4520-bbfe-42fd8d005b92

---

By the end of last semester, I was preparing for a deep dive into deployments, monitoring, maintenance, and the operational crossroads where Backend meets DevOps. My goal was to bridge my previous cloud experience—especially with GCP—with the modern stack we rely on at [JetRockets](https://www.linkedin.com/company/jetrockets/), such as [Kamal](https://kamal-deploy.org/), [AppSignal](https://www.appsignal.com/), [DigitalOcean](https://www.digitalocean.com/), and [Cloudflare](https://www.cloudflare.com/). When the syllabus dropped for my second semester in the [Artificial Intelligence program at UFRN](https://pes.imd.ufrn.br/pes/index), I was thrilled to spot MLOps (Machine Learning Operations) on the list. While the name might sound niche, [MLOps](https://en.wikipedia.org/wiki/MLOps) is essentially the application of strict software engineering and DevOps principles to machine learning, ensuring models aren't just local experiments, but are effectively deployed, monitored, and maintained in production.

<img width="640" height="460" alt="image" src="https://github.com/user-attachments/assets/ae6da9b6-b918-4b44-a468-b3cd89dcf641" />

Going in, I fully expected we'd immediately dive into server clusters, CI/CD pipelines, and configuration files. I'm glad I was wrong. The course actually kicked off with [Design Thinking](https://online.hbs.edu/blog/post/what-is-design-thinking). It forced us to step back from the terminal and discuss the importance of understanding the business problem, the user, and the real-world context before throwing infrastructure at a problem. To solidify this knowledge, my approach was to build a bridge between these MLOps concepts and my daily backend reality.

To bring these concepts out of the classroom and into reality, I needed a concrete project. I chose running analysis—a domain I know deeply after nine years in the sport. While I already had the core math mapped out in [an open-source gem](https://rubygems.org/gems/calcpace), I wanted to build a complete system to test these operational practices in the wild. The ultimate result of this effort is [Calcpace](https://calcpace.app/). Recently shipped to production, it is a fully-fledged web application offering running conversions, VO2 max estimates, race calendars, and pace predictions in 15 different languages. Under the hood, it serves as the perfect sandbox for merging backend and ML operations, powered by Ruby on Rails, PostgreSQL, Redis, Sidekiq, Kamal, AppSignal, DigitalOcean, and Cloudflare.

Here are some highlights of how the MLOps principles we studied mapped directly onto this project:

1. **Design Thinking and the Cost of an Error**: In ML, optimizing a mathematical metric means nothing if it doesn't solve a user's problem. For Calcpace, this meant studying existing market solutions and identifying exactly where runners struggle with pace and VO2 max calculations. It echoed the principles of [Domain-Driven Design, by Eric Evans](https://www.domainlanguage.com/ddd/blue-book/)—you have to align your software architecture with the business reality. In a running app, a 'bad prediction' isn't just a UI bug; it could mean an athlete pacing a marathon entirely wrong and hitting the wall.

2. **TDD as a Pipeline Foundation**: We revisited Test-Driven Development not just as a coding habit, but as a critical safeguard. Automated testing is the backbone of both standard software engineering and MLOps. In ML, you don't just test if the code compiles; you test if the data schema is valid and if the model's baseline performance holds. Solidifying the RSpec suite for Calcpace was essential to ensure the core math remained completely reliable as new features were added.

3. **Serving Models with the Rails Stack**: Ruby on Rails might not be the default poster child for Machine Learning, but it is a phenomenal wrapper for serving ML models via web APIs. With recent Rails versions adopting Kamal as the default deployment tool, containerized deployments have never been smoother. Kamal allows us to package the application in a reproducible Docker environment—a core tenet of MLOps—and push updates to DigitalOcean instances quickly and reliably.

4. **CI/CD and Workflow Automation**: Continuous Integration and Deployment are non-negotiable. While the course explored ML-specific tracking tools like Weights & Biases alongside GitHub Actions, I focused on mastering the latter for workflow automation. Every time a new version is released, GitHub Actions runs the test suite and seamlessly triggers a Kamal deployment. It’s a closed loop that ensures the live application is always perfectly synced with the repository.

5. **Beyond Uptime (Monitoring)**: The course introduced the LGTM stack (Looker, Grafana, Tempo, Mimir) for deep observability, which is standard for tracking infrastructure and data drift in ML. For the Calcpace Rails ecosystem, AppSignal paired with Cloudflare serves as our equivalent. It provides fantastic out-of-the-box APM, tracking request latency, background job queues, and error rates, ensuring we maintain a smooth user experience while giving us the dashboard visibility an MLOps mindset demands.

6. **Cloud & Storage Resilience**: We zoomed out to look at infrastructure as a whole—managing VMs, securing CI/CD secrets, and configuring scalable storage. I applied this by architecting the app on DigitalOcean and leveraging Cloudflare R2 buckets for efficient, S3-compatible object storage for static assets.

7. **DevOps in an AI-Scraped World**: Finally, exposing an API or web app today means dealing with a massive influx of automated traffic. I configured custom WAF (Web Application Firewall) rules to block malicious actors while ensuring the site remains fully accessible to legitimate crawlers. This balance is vital for SEO and for ensuring the content is properly indexed by modern AI search bots.

Ultimately, the final deliverable for this discipline isn't a static repository; it's a living, breathing product. MLOps is inherently a continuous cycle of monitoring, maintaining, and iterating. Calcpace is now live for runners globally, and we recently hit a massive milestone: the core Ruby gem just crossed 10,000 downloads. Looking ahead, the architecture is now primed for the next step—integrating real, predictive machine learning models into the Rails backend to provide even deeper performance insights for athletes.

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/cce45f91-8dce-41e1-92e1-2480d1f9dbd2" />

Images: "Running down the shadows" of Vinoth Chandar. [Open Verse, Creative Commons 2.0](https://openverse.org/image/b85eb957-92f7-43b9-8560-3922b2beb361) and MLops ([Wikipedia](https://en.wikipedia.org/wiki/MLOps))

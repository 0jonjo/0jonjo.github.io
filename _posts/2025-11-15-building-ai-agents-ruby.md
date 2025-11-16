---
layout: post
title: "Building Robust AI Agents with Ruby_LLM"
description: "A roundup of my two new technical posts on building robust AI in Ruby. Learn about intelligent Function Calling and building resilient, fault-tolerant clients"
tags: programming ruby ai english
categories: misc
youtubeId:
image: https://github.com/user-attachments/assets/c4129e7b-8815-451e-bea3-1fc5bfad6c24
---

I've been busy writing this month and wanted to share two new technical posts I published over on the JetRockets blog. Both are focused on building more capable and reliable AI applications in Ruby, and they complement each other well.

### First up: Building Intelligent AI Agents with Function Calling in Ruby

This post dives into one of the most exciting features of modern LLMs: Function Calling (also known as Tool Calling). This is what elevates a model from a simple text generator into an intelligent agent that can interact with real-world data and services.

In the article, I explore how you can use the ruby_llm gem to give your AI "tools" it can use to:

Access real-time, external data (like calling a live weather API).

Enforce your own internal business logic (like checking a user's membership status before granting access).

Extract structured JSON data directly from a user's request (for example, turning "a 3-day trip to Tokyo" into a structured itinerary).

If you're looking to build an AI that can do things rather than just talk, this post is for you.

Check it out here: [https://jetrockets.com/blog/building-intelligent-ai-agents-with-function-calling-in-ruby](https://jetrockets.com/blog/building-intelligent-ai-agents-with-function-calling-in-ruby)

### Next: Building a Resilient AI Client in Ruby with Stoplight and ruby_llm

Calling external AI services is powerful, but what happens when those services are slow, return errors, or go down completely? This post is all about building a resilient client that can handle these inevitable failures gracefully.

I introduce the Circuit Breaker pattern and show how to implement it using the stoplight gem. The core idea is to build a system that can:

Detect when a specific AI provider (like GPT-4o) is failing.

"Trip a circuit" to temporarily stop sending requests to that failing service.

Automatically and seamlessly failover to a backup model (like Gemini or a different GPT model), preserving the conversation history.

This one is all about ensuring your application remains stable and fault-tolerant, even when its external dependencies are having a bad day.

You can read the full post here: [https://jetrockets.com/blog/building-a-resilient-ai-client-in-ruby-with-stoplight-and-ruby_llm](https://jetrockets.com/blog/building-a-resilient-ai-client-in-ruby-with-stoplight-and-ruby_llm)

Together, these two articles provide a solid foundation for building AI applications in Ruby that are not only intelligent but also robust and reliable. I hope you find them useful!

<img width="756" height="525" alt="image" src="https://github.com/user-attachments/assets/c4129e7b-8815-451e-bea3-1fc5bfad6c24" />

Image: "Weaving Wires 2 by Hanna Barakat & Archival Images of AI + AIxDESIGN". [Better Images of AI, Creative Commons 4.0](https://betterimagesofai.org/)

---
layout: post
title: "Polished Ruby Programming: A book about fundamentals that landed right in my AI work"
description: "Reviewing the second edition of Jeremy Evans' book: what it covers, and the two chapters that answered questions I had been chewing on all year while shipping AI features in Rails."
tags: programming ruby ai english
categories: misc
youtubeId:
image: https://github.com/user-attachments/assets/489984c5-0dbc-40a5-af1e-f1838b7f41c7
---

A few weeks ago, someone from [Packt](https://www.packtpub.com/) reached out to me on LinkedIn. They had just released the second edition of [Polished Ruby Programming](https://code.jeremyevans.net/polished-ruby-programming.html) and offered to send me a copy if I was willing to share my thoughts on it. You know I enjoy good tech books and work heavily with Ruby, so here we are. When the ebook arrived, I started skimming through it, expecting a solid, standard refresher on best practices.

It turned out to be much more than that—and in a direction I completely didn't expect.

## What's inside

The scope is massive. Across 428 pages, [Jeremy Evans](https://code.jeremyevans.net/) covers core classes, variable and method design, error handling, code formatting, library and plugin architecture, metaprogramming, DSLs, testing, refactoring, deprecation, design patterns, concurrency, static versus duck typing, and optimization.

Evans is a Ruby committer and the maintainer of [Roda](https://roda.jeremyevans.net/) and [Sequel](https://sequel.jeremyevans.net/), so his opinions come with receipts. What I appreciated most is that almost nothing is handed down as an absolute rule. Even the SOLID principles arrive with a warning against applying them dogmatically. Every technique—a plugin system, a DSL, `method_missing`, static types—is presented as a trade-off. You are expected to decide for yourself, but you get a clear description of what each choice will cost you later. That's a rarity in programming books.

A lot of the content is highly valuable on its own. But two chapters genuinely stopped me in my tracks because they answered questions I'd been wrestling with all year while building AI features into production Rails apps.

Here are my two highlights.

## First highlight: Breaking circuits

This one hit close to home.

Chapter 5 is all about error handling: return values versus exceptions, designing APIs that are hard to misuse, fail-open versus fail-closed, retrying transient errors with backoff, and shaping exception hierarchies. Deep in the chapter, Evans builds a small circuit breaker from scratch: if a service fails three times in a minute, you stop calling it for a while, preventing every request from queuing up behind a provider that's already down.

Then he closes the section with the exact kind of pragmatic advice I wish more books gave:

> "If you need a circuit breaker for production code, you should probably use one of the many circuit breaker gems for Ruby instead of trying to implement a circuit breaker yourself, unless you have specific requirements not handled by an existing gem."

That is the exact conclusion I reached last year when writing [Building a Resilient AI Client in Ruby with Stoplight and ruby_llm](https://jetrockets.com/blog/building-a-resilient-ai-client-in-ruby-with-stoplight-and-ruby_llm) for the [JetRockets](https://www.linkedin.com/company/jetrockets/) blog. The goal was to detect when a model provider is failing, trip the circuit, fail over to a backup model, and keep the conversation history intact. I went with [Stoplight](https://github.com/bolshakov/stoplight) rather than rolling my own, for the exact reason Evans points out.

Still, reading the from-scratch implementation was incredibly useful. You end up understanding exactly what the gem handles for you, and more importantly, what it doesn't. This chapter also gave me an insight my own article was missing: separating permanent from transient errors is a structural decision that belongs in your exception hierarchy, not in a `rescue` clause bolted on later. A rate limit and a malformed request are fundamentally different failures, and only one of them deserves a retry. Until now, I'd been treating both simply as "the provider is having a bad day."

## Second highlight: Only mock what you cannot control

The testing chapter drops this rule:

> "If you must mock, only mock what you cannot control, such as calls to external APIs."

For standard HTTP clients, most of us just nod and move on. But try applying that to a feature built around a Large Language Model, and it becomes the definitive answer to a testing problem I'd been circling for weeks.

An AI model isn't a normal dependency. It's slow, it costs money on every call, and—the part that really wrecks a test suite—it's non-deterministic. Same input, different output. This usually leaves you with two bad options. Option A: Mock the entire pipeline, turning your tests into pure theater. They pass forever, asserting only that your own stubs were called, while real regressions walk straight through to production. Option B: Mock nothing, leaving you with a test suite that is flaky, painfully slow, and bills you on every CI run.

Evans' rule puts the boundary exactly where it belongs. The model provider API is the *only* thing I cannot control, so it is the *only* thing I stub. Everything else runs for real: prompt assembly, the batching loop over long content, the context management that keeps terminology consistent across chunks, the state machine of the background job, and saving the final result. If I break the batching logic, a test fails. That's exactly what tests are for.

I also learned two hard lessons by getting this wrong initially. First, stubbing the client isn't enough—you also have to pin the exact payload it hands back. Otherwise, the non-determinism just moves one layer down and your assertions start drifting. Second, this testing style severely punishes heavy inline setup. Once I moved the sample AI context into fixtures, the specs became readable and just as fast as any other model test in our suite.

The same chapter states the trade-off plainly, and it's a quote I'd gladly put on my wall:

> "Slower, reliable tests are better than fast tests that break without reason (false positives) and don't catch actual breakage (false negatives)."

## Zero hits

Out of curiosity, I eventually ran a search across the entire ebook for the terms "LLM", "AI", and "ChatGPT".

Zero hits. Not a single one in 428 pages.

And that is the part I keep coming back to. This isn't a book about AI, and that is precisely why it helped so much. A model provider is slow, occasionally unavailable, rate-limited, non-deterministic, and entirely outside your control. We already have decades of solid engineering practices for handling dependencies that behave exactly like that.

The only genuinely new thing is that AI failures are more convincing. A broken API simply returns a `503`. A broken model returns a fluent, confident, and nicely formatted wrong answer.

The architectural boundary is exactly where it always was. It just matters a lot more now.

## Who should read it

Intermediate and senior Ruby developers, without hesitation. If you're still learning the syntax, learn it somewhere else first—this book assumes you already write Ruby and want to write it better.

And if you maintain any long-lived applications, Chapter 12 earns the read all on its own. Evans argues that features should be treated as liabilities rather than assets, since every single one of them carries a permanent maintenance cost. Therefore, removing a feature is a net gain. It's the exact same conviction I wrote about a while back regarding [removing legacy code](/blog/2024/remocao-codigo/), just stated much more sharply than I managed at the time.

*Disclosure: Packt sent me a review copy of the book. The opinions here are entirely my own.*

<img width="640" height="333" alt="HannaBarakat -CambridgeDiversity FundPas(t)imesin the Computer Lab -640x333" src="https://github.com/user-attachments/assets/489984c5-0dbc-40a5-af1e-f1838b7f41c7" />

Image: "Pas(t)imes in the Computer Lab", Hanna Barakat & Cambridge Diversity Fund. [Better Images of AI, Creative Commons 4.0](https://betterimagesofai.org/images?artist=HannaBarakat&title=Pas%28t%29imesintheComputerLab)

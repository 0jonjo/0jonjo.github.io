---
layout: post
lang: en
locale: en_US
title: "Handling failed LLM calls in Rails with RubyLLM"
description: "What to do when an LLM call fails in a Rails background job: sort failures into transient, stochastic, deterministic and optional, place each retry at the right layer, roll back the conversation, count the spend and apply results atomically."
tags: programming ruby ai
categories: misc
youtubeId:
image: https://cdn.jetrockets.com/post/292/og_image/e732bb198decbc40e2d5caea34d2c4cf.png
---

Recently, I wrote an article about handling failed LLM calls in Rails with RubyLLM to [JetRockets blog](https://jetrockets.com/blog/handling-failed-llm-calls-in-rails-with-rubyllm). Here is the full article:

Your app translates a long document in batches, all in one conversation so the terms stay consistent. Halfway through, you ask the model to fix a heading it left in the original language, the provider times out, and since the fix is optional you move on. But that request is still in the conversation, unanswered, and it goes out with the next batch. A failed LLM call can leave a broken conversation, a bill for work you threw away or a half-written record behind, and sometimes it doesn’t even raise: it’s a 200 with an answer your code can’t use. In previous articles, we [**translated rich documents**](https://jetrockets.com/blog/translating-rich-documents-with-ai-using-ruby_llm) and added [**editable prompts and guardrails**](https://jetrockets.com/blog/editable-prompts-and-ai-guardrails-in-rails-with-ruby_llm) with RubyLLM. Now we’ll look at what to do when the call itself fails: try again, ask again, stop paying, or move on. All examples run on RubyLLM.

## Kinds of Failed Calls

A failed call is any LLM request whose result you can’t use, even when the provider returned a 200. Isn’t that just an exception to rescue? Not always, because the right response depends on what went wrong. Think of it like triage in an emergency room. Everyone comes through the same door, and the first job is deciding who needs what. We sort failures into 4 kinds:

1. **Transient:** the provider couldn’t answer, with a rate limit, a 503 or a timeout. Try again shortly.
2. **Stochastic:** the provider answered and your code rejected it, because the markup changed. Ask again, since the next answer will usually differ.
3. **Deterministic:** a retry would end the same way, like a bad request or a bug in how the result is applied. Stop, because the next attempt pays for all the work again.
4. **Optional:** the call was an improvement you can live without. Drop it and keep what you have.

The last one cuts across the others: any of the first 3 can hit a call you could live without.

The running example is a background job that translates a document. It splits the document into **segments**, small pieces of HTML with an id, groups them into batches, and sends every batch to the same conversation so terms stay consistent.

## Where Each Retry Lives

Retries can happen at 3 layers, and the bill usually grows with each one.

The **transport** layer belongs to RubyLLM. It retries rate limits, server errors, overloads, timeouts and connection failures, 3 times by default, with backoff and respect for `Retry-After`, so there’s no need to rebuild it. A 400 goes straight up, because the same request gets the same answer.

The **conversation** layer is a repair turn: send the rejected segments back and say what was wrong. It costs one call.

The **job** layer starts over and pays for every batch again, so it is the one to protect:

```ruby
# app/jobs/translate_document_job.rb
class TranslateDocumentJob < ApplicationJob
  NOT_WORTH_RETRYING = [
    RubyLLM::ConfigurationError, RubyLLM::ModelNotFoundError,
    RubyLLM::BadRequestError, RubyLLM::ContextLengthExceededError,
    RubyLLM::UnauthorizedError, RubyLLM::ForbiddenError, RubyLLM::PaymentRequiredError,
    Translator::CutOff, Applier::ShapeChanged, ActiveRecord::RecordInvalid
  ].freeze

  give_up = lambda do |job, error|
    Rails.error.report(error)
    DocumentTranslation.find_by(id: job.arguments.first)&.mark_failed!
  end

  retry_on StandardError, wait: :polynomially_longer, attempts: 3, &give_up
  retry_on RubyLLM::RateLimitError, wait: 1.minute, attempts: 3, &give_up
  discard_on(*NOT_WORTH_RETRYING, &give_up)

  def perform(translation_id)
    translation = DocumentTranslation.find(translation_id)
    return if translation.completed?

    DocumentTranslation::Run.new(translation).call
  end
end
```

Notice the order. Active Job checks handlers from the last one declared to the first, so everything specific must come after `retry_on StandardError`. Swap them and the catch-all retries everything.

The discard list holds the errors a new attempt can’t change: a bad request, a missing key, an answer cut off at the token limit or by a content filter, or a translated copy whose shape doesn’t match its source (`Applier::ShapeChanged`, a bug in our code, not the model’s). Everything else is retried, because an unknown error in a job usually deserves another try. The cost is that a 4xx we didn’t list, like a 404, gets retried too.

Rate limits get a longer wait. When `Retry-After` is longer than `retry_max_interval` (30 seconds by default), RubyLLM raises instead of waiting, and `:polynomially_longer` would burn all 3 attempts in about 20 seconds, paying for the earlier batches each time.

Only the job marks a translation as failed. If `DocumentTranslation::Run`, the class that drives the batches, did it halfway, the customer would see a failed status that the next attempt silently restarts.

Finally, the transport and the job multiply. With the defaults (a 5-minute `request_timeout` and 3 retries), one job attempt can hang for 20 minutes on a provider that keeps timing out, and 3 attempts make that about an hour. Set `request_timeout` to what one batch really needs, in a `RubyLLM.context` if other features need more, rather than lowering `max_retries`: every retry you remove sends more short outages to the job layer.

## The Conversation Remembers (Roll Back the Turn)

`chat.ask` adds your message to the history and only then calls the provider. If the call raises, the message stays in `chat.messages`. A single-call feature throws the chat away, so you never notice. Batches that share a conversation send that leftover message with every later request.

To see the effect, we translated a batch, placed a re-ask in the chat without an answer (the same state a failed `ask` leaves), and sent the next batch. `gpt-5.4-mini` ignored the leftover turn in 9 of 10 runs. In the other, it translated the leftover segments together with the batch, which our id check rejects. In all 10 clean runs, it answered exactly the batch.

The same thing happens in a hand-written failover loop that calls `chat.ask(prompt)` on one chat for each model: a model that fails leaves its copy of the prompt behind, and the next one receives it twice. Use `with_fallbacks`, which tries the next model inside the same generation and sends the prompt once. If every model fails, the turn still stays, so the rollback below still applies.

For the optional call, take a mark before asking and go back to it on failure:

```ruby
# app/models/translator.rb
class Translator
  class CutOff < StandardError; end

  def initialize(language:)
    @language = language
  end

  def call(segments)
    ask(segments, "Translate these segments:")
  end

  def retranslate(segments)
    reverting_on_failure do
      answers = ask(segments, "These are still in the source language. Translate them:")
      Validator.validate!(segments, answers)
      answers
    end
  end

  def repair(segments, problems)
    ask(segments, "These didn't pass our checks (#{problems.to_json}). Translate them again:")
  end

  private

  def ask(segments, instruction)
    payload = JSON.generate(segments.map { |segment| { id: segment.id, text: segment.text } })
    response = chat.ask("#{instruction}\n#{payload}")
    raise CutOff, response.finish_reason.to_s if response.max_tokens? || response.content_filtered?

    answers(response, segments)
  end

  # answers reads response.parsed and raises UnusableAnswer unless the ids match the batch.

  def reverting_on_failure
    mark = chat.messages.length
    yield
  rescue StandardError
    chat.messages = chat.messages.first(mark)
    raise
  end

  def chat
    @chat ||= RubyLLM.chat(model: "gpt-5.4-mini")
                     .with_instructions("Translate the text of each segment into #{@language}. Don't change the HTML tags.")
                     .with_schema(TranslationSchema)
  end
end
```

`TranslationSchema` comes from our translation article. `Validator` does too, with one change: a `problems` method collects what’s wrong with each segment (a blank answer or changed markup) instead of raising on the first one.

`chat.messages=` is the documented way to replace a transcript. Going back to a mark, instead of popping the last message, also removes an answer that arrived and then failed validation, so later batches don’t read it as context. `repair` has no rollback, because a failed repair fails the attempt and the next attempt builds a new chat. The rollback runs for a timeout and for a rejected answer alike, since in both cases we keep using the chat. After a successful repair we keep the whole exchange, because the corrected answer is the context we want.

The docs for a plain chat don’t spell out the history after a failed `ask`, so test this path after each upgrade. With `acts_as_chat`, the user message is saved first and a failed call removes only the empty assistant record (see `cleanup_after_failure` in the gem), so a row stays behind. There, `messages=` is the association writer and this snippet doesn’t carry over, so handle that row on the record instead.

## Count the Spend (Usage per Attempt)

A run that fails on its third batch already paid for the first 2. RubyLLM tracks each physical attempt and reports it through `usage.ruby_llm`, and wrapping the run in `RubyLLM.with_owner(@translation)` (2.2+) attributes every call inside it to that record:

```ruby
# config/initializers/ai_usage.rb
ActiveSupport::Notifications.subscribe("usage.ruby_llm") do |event|
  usage = event.payload
  next unless usage[:owner]

  AiUsage.create!(owner: usage[:owner], status: usage[:status], model: usage[:model],
                  input_tokens: usage[:tokens].input, cached_tokens: usage[:tokens].cache_read,
                  output_tokens: usage[:tokens].output, cost: usage[:cost].total)
rescue StandardError => error
  Rails.error.report(error)
end
```

The `rescue` is there because the event fires in the middle of the provider call. An exception in a subscriber would escape `chat.ask` after you’ve been billed for the answer. `tokens.input` doesn’t include cached input, so store `cache_read` too, especially when batches share a conversation.

A 503 retried by the transport leaves 2 rows, one `failed` and one `succeeded`, and a job that restarts adds its own, so summing by owner gives the reported tokens and cost of every attempt. If you ran RubyLLM’s 2.1 install or upgrade generator, the usage ledger stores these attempts in `ruby_llm_usages` with the owner, and you may not need the subscriber at all.

One detail is easy to miss: `succeeded` means the provider answered. An answer your validator threw away is still a `succeeded` row with its tokens, which is correct, because you were billed for it.

## Network First, Then One Transaction (Atomic Apply)

Every provider call happens before the transaction. Inside it, the copy and the status change commit together:

```ruby
# app/models/document_translation/run.rb
def write(answers)
  ActiveRecord::Base.transaction do
    copy = Document.create!(title: document.title, body_html: Applier.apply(document.body_html, answers))
    @translation.complete!(copy)
  end
end

# app/models/document_translation.rb
def complete!(copy)
  update!(status: "completed", copy:, error_message: nil)
  broadcast_status
end

def broadcast_status
  ActiveRecord.after_all_transactions_commit { broadcast_replace_later_to(self) }
end
```

A transaction around a network call would hold a database connection for minutes, and usage rows written inside it would roll back with it. The broadcast waits for the commit because otherwise the browser could be sent to a record that doesn’t exist yet, or never will.

## The Message the Customer Reads

A raw provider error can name the vendor, the model and your quota. Report the exception to your error tracker and store your own sentence:

```ruby
# app/models/document_translation.rb
def mark_failed!(message: "The translation failed. Please try again.")
  return if completed?

  update!(status: "failed", error_message: message)
  broadcast_status
end
```

The guard on the first line is there because 2 copies of a job can overlap, or an error can arrive after the commit. `give_up` must not mark a document as failed while the customer is already reading it. The guard doesn’t stop overlapping runs from each creating a copy; for that, lock the record inside the transaction.

## Putting It All Together

```ruby
# app/models/document_translation/run.rb
def call
  RubyLLM.with_owner(@translation) do
    answers = segments.each_slice(BATCH_SIZE).reduce({}) { |done, batch| done.merge(translate(batch)) }
    write(answers)
  end
end

def translate(batch)
  answers = @translator.call(batch)
  answers = answers.merge(retranslated(batch, answers))
  answers.merge(repaired(batch, answers))
end

def retranslated(batch, answers)
  untranslated = batch.select { |segment| answers[segment.id] == segment.text }
  return {} if untranslated.empty?

  @translator.retranslate(untranslated)
rescue StandardError => error
  Rails.error.report(error)
  {}
end

def repaired(batch, answers)
  problems = Validator.problems(batch, answers) # { segment id => what was wrong }
  return {} if problems.empty?

  rejected = batch.select { |segment| problems.key?(segment.id) }
  @translator.repair(rejected, problems).tap { |fixed| Validator.validate!(rejected, fixed) }
end
```

That’s the whole policy in 4 methods. The first call and the repair raise, so the job decides. The re-ask reports the error and carries on. The customer still gets the document, with one heading to fix by hand.

## Advantages

- **Lower spend on failure:** the errors you list stop after one attempt instead of paying for the document 3 times.
- **Clean state:** a failed optional call doesn’t stay in the conversation, and a failed write leaves no partial copy behind.
- **Visible cost:** every attempt is counted, including retries and answers you rejected.
- **Safe messages:** customers read your sentence, and your error tracker gets the details.

## When to Use This

Use this when a call is part of a multi-step job, when conversations carry across calls, or when results are written to your database. Skip most of it for a single call that fills a suggestion box: there’s no chat to roll back, and swallowing the error is the right answer when the feature has a degraded mode.

One limitation: restarting the job pays again for batches that already succeeded. For very long documents, saving progress per batch is worth its complexity, and [**Active Job Continuations**](https://jetrockets.com/blog/resilient-ai-workflows-with-activejob-continuable-in-rails) give you those checkpoints. We don’t do that here, because a resumed job would need the earlier answers saved and the conversation rebuilt, or it loses the terms the earlier batches settled.

## Best Practices

**Check `max_tokens?` before parsing:** a truncated answer is missing its end, so no parser can recover it, and a retry with the same limit tends to stop in the same place. A chat response blocked by a content filter doesn’t raise either, so check `content_filtered?` as well.

**Mind what `RubyLLM::Error` doesn’t cover:** `ConfigurationError` and `ModelNotFoundError` inherit from `StandardError`, so `rescue RubyLLM::Error` misses them. Timeouts and connection failures arrive as `RubyLLM::ConnectionError`, which it does catch, and `timeout?` tells the two apart.

**Treat missing usage as unknown:** a request that timed out may still have been billed, so RubyLLM records `nil` rather than zero, and a SQL `SUM` skips it. Requests the provider refused, or that never reached it, record 0.

**Assert on the request body:** stub the provider with a failure, then a success, and check the body of the second request. A leftover turn shows up there, even when the final response looks right.

**Run the failure paths against a real provider too:** a stub proves what you send, not how a given model reacts to it.

## Conclusion

Back to the document from the start. The heading fix timed out, the rollback took the request out of the conversation, and the next batch went out clean. The customer got the translation with one heading to fix by hand, and every attempt showed up in the usage. Most of what made that work is ordinary Rails: a job that knows when to give up, a transaction, a message the customer can read. Only two things are specific to LLMs: a failed `ask` stays in the history, and a failed attempt can still cost you. And when a whole vendor is down, every new request still tries it first. To skip it, you need a circuit breaker, which we covered when [**migrating to RubyLLM 2.0**](https://jetrockets.com/blog/migrating-to-rubyllm-2-0-what-we-deleted-and-what-came-back).

## Additional Resources

- 📄 [**RubyLLM documentation: error handling**](https://rubyllm.com/error-handling/)
- 📄 [**RubyLLM documentation: what’s new in 2.2**](https://rubyllm.com/whats-new-in-2-2/)
- 📄 [**Instructor: retry logic**](https://python.useinstructor.com/concepts/retrying/), on counting transport retries apart from repair attempts
- 📄 [**Active Job basics: retrying or discarding failed jobs**](https://guides.rubyonrails.org/active_job_basics.html#retrying-or-discarding-failed-jobs)

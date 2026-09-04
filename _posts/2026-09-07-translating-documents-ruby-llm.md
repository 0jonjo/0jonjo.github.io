---
layout: post
title: "Translating rich documents with AI using ruby_llm"
description: "A production-ready Ruby pipeline that translates whole documents with LLMs while keeping markup, links and structure intact: extract, batch, translate, validate, apply."
tags: programming ruby ai english
categories: misc
youtubeId:
image: https://cdn.jetrockets.com/post/284/og_image/190a3dd3214cdb2b5dffd7b4cffd0560.png
---

Recently, I wrote an article about translating rich documents with AI using ruby_llm to [JetRockets blog](https://jetrockets.com/blog/translating-rich-documents-with-ai-using-ruby_llm). Here is the full article:

LLMs are genuinely good at translation. Send a model a few paragraphs and you get a natural, fluent version back, it feels almost trivial. Translating an entire document is a different job. The formatting, the images, the links and internal references all have to survive the trip, and the model has no particular commitment to any of them. In practice, a model will drop tags, leave whole lists untranslated, and, every now and the, swap a letter "e" for an invisible Cyrillic look-alike that no human reviewer would ever catch. This article is about the distance between those two tasks — and the pipeline that closes it: extract, batch, translate, validate, apply, built in Ruby with ruby_llm.

The gem earns its place in every stage that touches a model by providing:

- A unified API across OpenAI, Gemini, and Anthropic.
- Stateful chats that turn segmented batches into a single conversation.
- Schema constraints that return parsed Ruby hashes instead of raw strings.
- Granular control over temperature and reasoning effort per request.

Everything else in the pipeline — extraction, validation, and assembly — is plain Ruby.

## Why a Pipeline?

A whole-document translation can fail in three ways: the model hallucinates content, it corrupts the markup, or the request times out. None of these should leave a half-translated document behind. That is why we treat translation as five ordered stages:

- Extract: Decomposes the document into translatable segments.
- Batch: Groups them under a character size budget.
- Translate: Sends each batch to the model under a schema constraint.
- Validate: Checks the answers against the source before persisting.
- Apply: Writes the translated text onto a fresh copy of the document.

This last step is pure and LLM-free, because everything fallible has already happened. The pipeline doesn't care where segments come from. We'll build it for HTML, but the same contract works for .docx files, where paragraphs are `w:p` nodes inside the zipped XML, and for PDFs at extraction time (text runs via `pdf-reader`). One honest caveat: with PDFs only the text layer travels, so you get translated text back, not a rebuilt PDF.

## Extracting Segments

Three gems do all the heavy lifting:

```ruby
# Gemfile
gem "ruby_llm"
gem "schematist"
gem "nokogiri"
```

The atom of the pipeline is a segment: one translatable string with a stable id.

```ruby
Segment = Data.define(:id, :text)

class TextExtractor
  PROSE = "p, h1, h2, h3, h4, li, blockquote"

 # Skip empty editor artifacts and extract only leaf nodes to prevent double translation.
  def self.segments(html)
    nodes = Nokogiri::HTML5.fragment(html).css(PROSE).reject { |node| node.at_css(PROSE) }
    nodes.each_with_index.filter_map do |node, index|
      Segment.new(id: index.to_s, text: node.inner_html) unless node.text.strip.empty?
    end
  end
end
```

One warning from the field: the prose selector is per-source, even if the rest of the pipeline is universal. The first real blog we pointed this at returned almost nothing. Its body was Trix (Action Text) output, where every paragraph is a bare `<div>` inside `div.trix-content`, not a `<p>`. Know your source's dialect before trusting your selector.

Text that lives outside the body joins the same stream under namespaced ids: a `MetadataExtractor` emitting `meta-title` and `meta-summary` segments, for example. Same validator, same applier; this is what makes the pipeline fit any document type.

## The Schema Contract

The natural output shape would be a JSON object keyed by segment id. Structured output can't express that: the schema's key set must be known when it is declared, and our ids only exist per request. We declare it with schematist (ruby_llm-schema renamed at 1.0: same DSL, new namespace). One seam worth knowing: schematist emits a bare JSON Schema document, and ruby_llm 1.16 forwards it to the provider untouched — $schema and title headers included. OpenAI currently  tolerates them, but your schema's name silently degrades to "response" in the request, and that tolerance is undocumented, ruby_llm's main branch already strips both headers. A  six-line base class builds the envelope with_schema expects:

```ruby
class LLMSchema < Schematist::Schema
    # ruby_llm unwraps this shape: `name` names the schema, `schema` is the document.
  def to_json_schema
    document = super
    { "name" => document["title"], "schema" => document.except("$schema", "title") }
  end
end
```

The schema itself is plain DSL:

```ruby
class TranslationSchema < LLMSchema
  array :translations, description: "Exactly one entry per source segment, same ids, any order." do
    object do
      string :id,   description: "The segment id, copied verbatim from the input."
      string :text, description: "The translation. If the input contained HTML, the same " \
                                 "markup with only human-readable text translated."
    end
  end
end
```

Notice the descriptions: they aren't documentation, they're prompt surface. The model reads them.

## The Translator

The Translator owns one chat for the whole document, and asks it once per batch (batching itself comes in a moment):

```ruby
class Translator
  # Matches the full model's output via strict prompting
  MODEL = "gpt-5.4-mini"

  def initialize(language:)
    @language = language
  end

  def call(segments)
    payload = JSON.generate(segments.map { |s| { id: s.id, text: s.text } })
    response = chat.ask("Translate these segments:\n#{payload}")
    raise "unparseable response: #{response.content.class}" unless response.content.is_a?(Hash)
    translations = response.content.fetch("translations")
    verify_ids!(segments, translations)
    translations.to_h { |t| [t.fetch("id"), t.fetch("text")] }
  end

  private

  def chat
    # Deterministic output (copying, not inventing). with_params, not with_temperature:
    # ruby_llm 1.16 normalizes with_temperature to 1.0 for the whole gpt-5 family
    # (fixed upstream in #719, unreleased); with_params merges after that normalization.
    @chat ||= RubyLLM.chat(model: MODEL)
                     .with_params(temperature: 0)
                     .with_thinking(effort: :none)
                     .with_instructions(system_prompt)
                     .with_schema(TranslationSchema)

  end

  def system_prompt
    <<~PROMPT
      You are translating the readable text of a document into #{@language}.
      Reproduce any HTML markup exactly: every tag and attribute must reappear unchanged;
      only human-readable text (including alt and title attribute values) is translated.
      Keep proper nouns, brand names and code identifiers in their original form.
      Return exactly one entry per source segment with the same id, copied verbatim.
    PROMPT
  end

  def verify_ids!(segments, translations)
    expected = segments.map(&:id)
    answered = translations.map { |t| t.fetch("id") }
    raise "ids answered more than once" if answered.tally.any? { |_, n| n > 1 }
    raise "ids do not match the batch" unless answered.sort == expected.sort
  end
end
```

`verify_ids!` runs per batch on purpose: a batch that echoes an id from another batch would silently overwrite that batch's answer and still look complete after the merge. Two provider notes worth knowing. GPT-5-family models only accept a `temperature` while reasoning is off — at any higher effort the API rejects the request — which suits us fine: for a copy task, both knobs point the same way. One catch in the gem, though: ruby_llm 1.16 — the current release — normalizes with_temperature to 1.0 for every gpt-5 model by id alone, logging the swap only at debug level. The blanket rule was fixed upstream the day after 1.16.0 shipped but hasn't been released; until it is,  with_params(temperature: 0) merges into the payload after the normalization and reaches the API. And reasoning enums differ per provider:

## The Validation Gate

The promise of the pipeline is that the output is the same document, translated — same tree, same markup, new prose. The validator is what turns that promise into a hard gate:

```ruby
class Validator
  class Invalid < StandardError; end

  # Compare presence only, as the model is expected to translate these values.
  PROSE_ATTRIBUTES = %w[alt title caption].freeze

  def self.validate!(segments, translated_by_id)
    segments.each do |segment|
      text = translated_by_id.fetch(segment.id)
      raise Invalid, "blank translation for #{segment.id}" if text.strip.empty?
      raise Invalid, "markup changed for #{segment.id}" unless tags(segment.text) == tags(text)
      raise Invalid, "attributes changed for #{segment.id}" unless attributes(segment.text) == attributes(text)
    end
  end

  def self.tags(html)
    Nokogiri::HTML5.fragment(html).css("*").map(&:name).tally
  end

  # Tag counts miss injected styles or scripts; validate every attribute explicitly.
  def self.attributes(html)
    Nokogiri::HTML5.fragment(html).css("*").flat_map do |element|
      element.attribute_nodes.map do |attribute|
        value = attribute.value unless PROSE_ATTRIBUTES.include?(attribute.name)
        [element.name, attribute.name, value]
      end
    end.tally
  end
end
```

Two design decisions live here, both tested by real content:

1. Strengthen the prompt, never relax the validator: a legacy editor once split a single sentence across three `<p>` tags. The model merged them into a better paragraph, but the validator correctly rejected it (16 tags ≠ 18 tags). You will be tempted to tolerate "benign" tag-count drift. Don't. We added a prompt rule instead (*"never merge or split paragraph boundaries"*), dropping the failure rate to zero. A relaxed validator is a blind spot for dropped links; a prompt rule is a targeted fix.
2. Tolerances must be explicit: the `PROSE_ATTRIBUTES` exemption list is where deliberate tolerance lives. Values like `alt` and `title` are prose; the model *should* change them. Later, we added `caption` to support Action Text image galleries. Every tolerance is an explicit decision, not an accident.

After validation, applying is boring on purpose. Re-walk a fresh copy of the document with the same deterministic query the extractor used, and write translations by position:

```ruby
class Applier
  def self.apply(html, translated_by_id)
    copy = Nokogiri::HTML5.fragment(html)
    copy.css(TextExtractor::PROSE)
        .reject { |node| node.at_css(TextExtractor::PROSE) }
        .each_with_index do |node, index|
          node.inner_html = translated_by_id.fetch(index.to_s) if translated_by_id.key?(index.to_s)
        end
    copy.to_html
  end
end
```

For documents that live in a database, this step runs inside one transaction, with every network call already behind us, so a failed run leaves nothing behind.

## Batching: Turns of One Conversation

Large documents don't fit one call, so segments are grouped under a character budget, always at segment boundaries, never mid-sentence:

```ruby
class Batcher
  MAX_BATCH_CHARS = 20_000

  def self.batches(segments)
    return [] if segments.empty?

    segments.each_with_object([[]]) do |segment, acc|
      current = acc.last
      if current.any? && current.sum { |s| s.text.length } + segment.text.length > MAX_BATCH_CHARS
        acc << [segment]
      else
        current << segment
      end
    end
  end
end

html = document.body_html
segments = TextExtractor.segments(html)
translator = Translator.new(language: "pt-BR")

translated = Batcher.batches(segments).reduce({}) do |acc, batch|
  acc.merge(translator.call(batch))
end

Validator.validate!(segments, translated)
translated_html = Applier.apply(html, translated)
```

A real segment through the whole pipeline — this one borrowed from NASA's Artemis II press kit — translating into Brazilian Portuguese:

```ruby
# Input:  <p>Orion will carry the crew about <a href="/mission-profile"
#         title="Mission profile">4,000 miles</a> beyond the far side of the Moon.</p>
# Output: <p>Orion levará a tripulação a cerca de <a href="/mission-profile"
#         title="Perfil da missão">4.000 milhas</a> além do lado oculto da Lua.</p>
```

The markup is byte-identical, the `href` untouched, the `title` translated, the number localized ("4,000" became "4.000"), and Orion stays Orion — the contract, visible in one line.

Why 20k characters and not one giant call? We measured it: doubling the batch size to collapse two calls into one saved 0.7 seconds — roughly the per-call overhead of the avoided request, because generation time itself tracks *output* tokens, not call count. Meanwhile the single giant request sits closer to the client timeout and raises the stakes of any one failure to the whole document.

The interesting part is that batches share one conversation. Independent calls might translate the same term two different ways. Making batch *N* a turn that still sees batches *1..N−1* fixed terminology flips in our tests. (For fun, we ran the pipeline over NASA's Artemis II press kit: 236 segments, two batches, one conversation, 35 seconds.)

**The key insight:** We tested *pruning* old turns to save money on long documents. It cost 24% more, because dropping middle turns invalidates the provider's cached prefix (automatic on OpenAI; Anthropic only caches blocks you explicitly mark), forcing every later call to pay full price. Carrying the full conversation history is nearly free (about 1% extra cost, zero extra latency) and produces strictly better translations. One conversation also has a ceiling, though: every batch re-sends the whole history, so a document large enough to overflow the context window fails on its last batches — after the earlier ones were paid for. Very large documents deserve a chat reset every K batches.

One operational note: none of this belongs in a request cycle. A large document takes minutes of wall clock, so the run lives in a background job with a bounded retry count — and that retry is doing double duty. Most of the model's failures are stochastic: a validation rejection that fails the job usually passes on the re-roll, so the same mechanism that survives a provider hiccup also cleans up the model's occasional bad day.

## Lessons from Production

Three lessons survived contact with real pages and are worth stealing even if you never build this exact pipeline.

**Guard with allowlists, not denylists.** Invisible-homoglyph defects (random foreign characters spliced into the output) happen. We added a validator allowing only the target language's alphabet, Latin, and characters present in the source. It immediately caught a foreign alphabet a denylist would have missed.

**A cheap model's "regression" is often a prompt gap.** A smaller model tier failed our place-name rules inconsistently. Rewriting the rule to explicitly name the failure mode (*"naming the same place two different ways is wrong"*) fixed it. The small model matched the large one exactly, at a third of the cost and 40% faster.

**Keep reasoning off for transformation tasks.** With the source text already in hand, there is nothing to derive. Enabling reasoning just adds latency and wastes tokens without improving the prose.

## Trade-offs: When (and When Not) to Use It

This architecture provides four structural guarantees:

- **Atomicity:** A failure at any stage leaves the source untouched and leaves no partial copies behind.
- **Deterministic Guards:** Markup fidelity and alphabet checks don't depend on the model having a good day.
- **Testability:** Extract, batch, validate, and apply are pure functions you can unit-test without API calls.
- **Portability:** The provider abstraction means the pipeline runs on OpenAI, Gemini, or Anthropic with a single model swap.

This pipeline earns its complexity when documents have structure worth preserving (CMS pages, knowledge bases, product catalogs, anything where a broken link or a dropped image is a real defect) and when volume makes human re-assembly impractical. It is the wrong tool for a paragraph of plain text (just ask the model), for certified or legal translation where a human must own every word, and for real-time content where the seconds-per-page latency doesn't fit.

## Conclusion

The model was never the hard part. Building a trustworthy document translator turned out to be classic engineering around a fluent-but-unreliable core: decompose into verifiable units, constrain the output shape, validate deterministically, apply atomically, and let measurements, not intuition, pick the model, the batch size and the conversation strategy. The pleasant surprise of doing this in Ruby is how little glue it takes: ruby_llm handles providers, schemas and conversations, and everything else is a hundred lines of the same plain objects we write every day.

## Additional Resources

- 📁 [**ruby_llm gem on GitHub**](https://github.com/crmne/ruby_llm)
- 📁 [**schematist on GitHub**](https://github.com/crmne/schematist)
- 📄 [**RubyLLM documentation — structured output**](https://rubyllm.com/chat#getting-structured-output)

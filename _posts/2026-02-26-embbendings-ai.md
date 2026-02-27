---
layout: post
title: "Building Semantic Search with AI and Vector Embedding in Rails"
description: "Creating semantic search with vector embeddings in Ruby using the ruby_llm gem and PostgreSQL's pgvector extension."
tags: programming ruby ai english
categories: misc
youtubeId:
image: https://github.com/user-attachments/assets/22ee97d1-ed97-4db9-8bbb-700d9c94f83d
---

This month, I wrote an article on building semantic search with vector embeddings in Ruby using the ruby_llm gem and PostgreSQL's pgvector extension to [JetRockets blog](https://jetrockets.com/blog/building-semantic-search-with-ai-and-vector-embedding-in-rails). Here is the full article:

Traditional keyword search is fundamentally limited—it can only match exact words or their basic variations. Search for "customer pain points" and you'll miss documents titled "User Frustrations" or "Client Challenges," even though they're semantically identical. This limitation becomes critical when managing large document repositories where users need to find information based on meaning, not memorized keywords. The solution is semantic search using vector embeddings, which represent text as mathematical vectors that capture conceptual similarity. In previous articles, we explored how ruby_llm simplifies working with AI providers for [function calling](https://jetrockets.com/blog/building-intelligent-ai-agents-with-function-calling-in-ruby) and [resilient architectures](https://jetrockets.com/blog/building-a-resilient-ai-client-in-ruby-with-stoplight-and-ruby_llm). Now we'll leverage ruby_llm's embedding capabilities to build semantic search with PostgreSQL's pgvector extension—creating a system that finds "churn analysis" when users search for "why customers leave."

## What Are Vector Embeddings?

Think of vector embeddings as a way to translate text into the language of mathematics—or as some describe it, "searching by vibes" rather than exact keywords. Each piece of text becomes an array of numbers (a "vector") where similar meanings produce similar numbers. The embedding model has learned, through training on billions of texts, that certain concepts cluster together in this mathematical space.

We'll use OpenAI's `text-embedding-3-small` (1,536 dimensions) for its solid performance-to-cost ratio, but ruby_llm also supports other models: Gemini's `text-embedding-004` (768 dimensions), Voyage AI's models, or even local alternatives like `all-MiniLM-L6-v2` via Ollama for privacy-sensitive applications. The choice depends on your budget, latency requirements, and whether you prefer cloud or self-hosted solutions.

For example:
- "customer churn analysis" → `[0.234, -0.891, 0.456, ...]`
- "user retention study" → `[0.221, -0.883, 0.449, ...]` (very close!)
- "quarterly revenue report" → `[-0.678, 0.234, -0.123, ...]` (very different)

When a user searches for "why are customers leaving", the system converts this query into a vector and finds documents with similar vectors—automatically surfacing reports about "churn factors" and "cancellation reasons" without requiring exact keyword matches.

## Setting Up ruby_llm for Embeddings

Just as we used ruby_llm for function calling in our previous article, we can use it for embeddings too. The gem provides a unified interface with consistent error handling and automatic retries:

```ruby
# config/initializers/ruby_llm.rb
require "ruby_llm"

RubyLLM.configure do |config|
  config.openai_api_key = Rails.application.credentials.dig(:openai, :api_key)
end

# Create a reusable client instance
LLM_CLIENT = RubyLLM
```

Now create a wrapper that handles the embedding API calls:

```ruby
# app/services/embedding_service.rb
class EmbeddingService
  MODEL = "text-embedding-3-small"
  DIMENSIONS = 1536

  def self.embed(texts)
    texts = Array(texts)
    return [] if texts.empty?

    # ruby_llm handles the API call automatically
    result = LLM_CLIENT.embed(texts, model: MODEL, dimensions: DIMENSIONS)
    result.vectors
  end
end
```

That's it! Using ruby_llm gives us the same benefits we saw in the function calling article: provider abstraction, automatic error handling, and consistent behavior across different AI services.

## Setting Up pgvector for Storage

Why pgvector? If you're already running PostgreSQL (and many Rails apps are), pgvector lets you store and search vectors without adding new infrastructure. No separate vector database to maintain, no data synchronization issues, and transactions work normally—your embeddings live right next to your application data. The [pgvector extension](https://github.com/pgvector/pgvector) adds specialized vector types and HNSW indexing for sub-100ms similarity searches across millions of vectors.

Before storing embeddings, enable the extension:

```ruby
# db/migrate/create_documents.rb
class CreateDocuments < ActiveRecord::Migration[8.0]
  def change
    enable_extension "vector"

    create_table :documents do |t|
      t.string :title, null: false
      t.text :content, null: false
      t.references :user, null: false, foreign_key: true
      t.timestamps
    end

    create_table :document_chunks do |t|
      t.references :document, null: false, foreign_key: true
      t.integer :chunk_index, null: false
      t.text :content, null: false
      t.vector :embedding, limit: 1536
      t.timestamps
    end

    # HNSW index for fast nearest-neighbor search
    add_index :document_chunks, :embedding, using: :hnsw, opclass: :vector_l2_ops
  end
end
```

The HNSW (Hierarchical Navigable Small World) index creates a graph structure that allows fast approximate nearest-neighbor search—essential for sub-100ms queries across thousands of vectors.

## Basic Pattern: Chunking Documents

Here's the challenge: documents can be very long, but embedding models work best on focused text segments (500-2000 characters). Embed an entire 50-page document and you'll get a diluted embedding that doesn't capture nuances. The solution is chunking with overlap. The overlap (typically 10-20%) prevents losing context at boundaries:

```ruby
# app/models/document.rb
class Document < ApplicationRecord
  CHUNK_SIZE = 1_000
  OVERLAP_SIZE = 200

  has_many :document_chunks, dependent: :destroy

  def generate_chunks!
    step = CHUNK_SIZE - OVERLAP_SIZE
    chunks = []

    # Split content into overlapping chunks
    0.step(content.length - 1, step) do |offset|
      chunk = content[offset, CHUNK_SIZE]
      chunks << chunk if chunk.present?
    end

    return if chunks.empty?

    # Generate embeddings for all chunks in one call
    embeddings = EmbeddingService.embed(chunks)

    ActiveRecord::Base.transaction do
      document_chunks.destroy_all

      chunks.zip(embeddings).each_with_index do |(chunk, embedding), index|
        document_chunks.create!(
          chunk_index: index,
          content: chunk,
          embedding: embedding
        )
      end
    end
  end
end
```

Notice how we generate embeddings for *all* chunks in a single call. Instead of making 50 separate API requests for a 50-chunk document, we make just one. The ruby_llm gem handles the batching automatically, just like it does with function calling.

The character-based approach above works, but cutting mid-sentence degrades semantic meaning. A better strategy splits on sentence boundaries using `text.scan(/[^.!?]+[.!?](?:\s+|$)/)`, accumulating sentences until reaching the size limit. This preserves complete thoughts while maintaining consistent chunk sizes—particularly valuable for technical documentation where sentence context matters.

## Advanced Pattern: Hybrid Scoring (Semantic + Temporal)

Pure semantic search has a problem: outdated documents with perfect semantic matches outrank recent documents with good matches. When users search for "current market size", they might get a 2023 report instead of the fresh 2025 one. The solution is hybrid scoring—combine semantic similarity (70%) with recency (30%):

```ruby
# app/models/document_chunk.rb
class DocumentChunk < ApplicationRecord
  belongs_to :document
  has_neighbors :embedding, dimensions: 1536

  attr_accessor :search_score

  def self.search_by_semantics(query, user:, limit: 20, threshold: 0.8)
    # Convert query to embedding using ruby_llm
    query_embedding = EmbeddingService.embed(query).first

    # Get more candidates than needed for scoring
    neighbors = joins(:document)
                  .where(documents: { user: user })
                  .nearest_neighbors(:embedding, query_embedding, distance: "cosine")
                  .limit(limit * 5)

    # Calculate hybrid scores
    scored_neighbors = neighbors.map do |chunk|
      semantic_similarity = 1 - (chunk.neighbor_distance / 2.0)
      recency_score = calculate_recency_score(chunk.created_at)

      # 70% semantic, 30% recency
      chunk.search_score = (semantic_similarity * 0.7) + (recency_score * 0.3)
      chunk
    end

    # Filter and return top results
    scored_neighbors
      .select { |c| c.neighbor_distance < threshold }
      .sort_by { |c| -c.search_score }
      .first(limit)
  end

  def self.calculate_recency_score(created_at)
    age_in_days = (Time.current - created_at) / 1.day
    return 1.0 if age_in_days <= 30  # Recent: full score
    return 0.7 if age_in_days > 365  # Old: 30% penalty
    1.0 - ((age_in_days - 30) / 335.0) * 0.3  # Linear decay
  end
end
```

The key insight: we fetch `limit * 5` candidates first, then score and filter. Why? A chunk might be the 50th-best semantic match but jump into the top 5 after adding recency. By casting a wider net initially, we don't miss valuable recent documents.

## Putting It All Together

Now let's see the complete flow from upload to search:

```ruby
class DocumentsController < ApplicationController
  def create
    document = current_user.documents.create!(
      title: params[:title],
      content: params[:content]  # or extract from uploaded file
    )

    # Generate chunks and embeddings asynchronously
    GenerateEmbeddingsJob.perform_later(document.id)

    render json: { id: document.id, status: "processing" }
  end

  def search
    results = DocumentChunk.search_by_semantics(
      params[:query],
      user: current_user,
      limit: 10,
      threshold: 0.75
    )

    render json: {
      query: params[:query],
      results: results.map do |chunk|
        {
          document_title: chunk.document.title,
          excerpt: chunk.content[0..300],
          relevance_score: chunk.search_score.round(2)
        }
      end
    }
  end
end
```

The architecture is completely asynchronous where it matters. Document uploads don't block waiting for embeddings—users get immediate feedback, and embeddings are generated in the background. Meanwhile, searches are fast (typically under 100ms) because pgvector's HNSW index does the heavy lifting.

## Advantages of This Approach

The benefits go far beyond "better search." The system demonstrates **semantic understanding**—users can search for "why customers switch providers" and find reports titled "Competitive Migration Patterns," even though those exact words don't appear in the query.

From an engineering perspective, the architecture is production-ready: **context preservation** through overlapping chunks means no information is lost at boundaries, **temporal awareness** via hybrid scoring ensures recent insights aren't buried, and **ruby_llm** provides the same advantages we explored in previous articles—automatic retry logic, consistent error handling, and clean abstractions for AI interactions.

## When to Use Semantic Search

Semantic search shines when users need to find documents by meaning rather than exact keywords—scenarios like customer support knowledge bases, research archives, or legal document discovery where synonyms and conceptual similarity matter. It's especially valuable powering AI features: chatbots citing documents, assistants surfacing relevant research, or agents (using ruby_llm's function calling) that search intelligently.

However, skip it for small document sets (<100 documents) with well-structured content, cases requiring exact phrase matching (legal contracts), or real-time updated content where the 1-2 second embedding latency is problematic. Traditional keyword search with proper indexing often suffices for simpler use cases.

## Best Practices

Through implementing semantic search systems, several best practices have emerged:

**Always Use Overlapping Chunks:** The 200-character overlap prevents context loss at boundaries. Resist the temptation to eliminate overlap to save on storage—it's a false economy that degrades search quality.

**Batch Your Embeddings:** Generate embeddings for all chunks in a single call to `EmbeddingService.embed(chunks)`. Process 20-chunk documents with one request instead of 20, reducing latency by 95% and API costs proportionally.

**Tune Thresholds Per Use Case:** The 0.8 default threshold works for most cases, but use 0.85 for mission-critical searches (where precision matters) and 0.75 for exploratory searches (where recall matters). Monitor your analytics and adjust accordingly.

**Async Everything That Can Be Async:** Never block user requests waiting for embeddings. Generate them in background jobs and show a "processing" indicator. Users can continue working while documents are indexed.

## Conclusion

Transitioning from keyword-based to semantic search transforms how users interact with your data, and as we've seen, it doesn't require adopting entirely new infrastructure. By keeping our architecture resilient with PostgreSQL and pgvector, we avoided the overhead of maintaining a separate vector database. We then used ruby_llm to abstract the complexity of interacting with AI providers, ensuring our API calls are batched and fault-tolerant.

Finally, we solved the real-world UX challenges of AI search by implementing overlapping text chunks to preserve context, and hybrid scoring to balance semantic accuracy with temporal relevance. The magic of modern AI tools in the Rails ecosystem is that they allow us to build highly sophisticated features—like "searching by vibes"—while relying on the same pragmatic, robust engineering principles we use every day.

## Additional Resources

- 📄 [Embeddings: What they are and why they matter](https://simonwillison.net/2023/Oct/23/embeddings/) by Simon Willison
- 📄 [What are embeddings?](https://vickiboykis.com/what_are_embeddings/) by Vicki Boykis
- 📁 [pgvector GitHub Repository](https://github.com/pgvector/pgvector)
- 📁 [neighbor gem for Rails](https://github.com/ankane/neighbor)
- 📁 [ruby_llm gem on GitHub](https://github.com/crmne/ruby_llm)

Image: "Word Search" by Morgan Frederick. [Open Verse, Creative Commons 2.0](https://openverse.org/image/79fc1531-5557-43e8-8314-0ef15b6c36b9)

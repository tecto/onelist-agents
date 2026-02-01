# Searcher Agent

The Searcher Agent provides semantic search capabilities across Onelist entries using embeddings and intelligent reranking.

## Status: ✅ Active

## Overview

Searcher enables users to find relevant information using natural language queries rather than exact keyword matches. It combines vector similarity search with LLM-powered reranking for high-quality results.

## Architecture

```
Query → QueryReformulator → Embedding → TwoLayerSearch → Reranker → Results
           ↓                    ↓              ↓
    Expand/refine query   Generate vector   Semantic search
                                               + keyword
```

## Components

### 1. Query Reformulator
- Expands ambiguous queries
- Generates multiple search variations
- Handles typos and synonyms

### 2. Chunker
- Splits long content into searchable chunks
- Preserves context at chunk boundaries
- Handles multiple content types (markdown, plain text, code)

### 3. Embedding Job (Oban Worker)
- Async embedding generation
- Batch processing for efficiency
- Automatic retry on failures

### 4. Two-Layer Search
- **Layer 1:** Fast vector similarity (pgvector)
- **Layer 2:** Keyword/BM25 hybrid
- Combines scores for best of both worlds

### 5. Reranker
- Uses Cohere's rerank API
- Scores retrieved results by relevance
- Filters low-confidence matches

### 6. Verifier
- Validates search results
- Checks for stale or deleted content
- Ensures permission compliance

## Configuration

```elixir
# config/config.exs
config :onelist, Onelist.Searcher,
  embedding_model: "voyage-3-lite",
  embedding_dimensions: 1024,
  rerank_model: "rerank-v3.5",
  max_chunks_per_entry: 50,
  chunk_size: 512,
  chunk_overlap: 50
```

## Usage

```elixir
# Basic search
{:ok, results} = Onelist.Searcher.search(user_id, "meeting notes from last week")

# With options
{:ok, results} = Onelist.Searcher.search(user_id, query,
  limit: 10,
  threshold: 0.7,
  include_archived: false
)
```

## Providers

| Provider | Use Case | Status |
|----------|----------|--------|
| Voyage AI | Embeddings | ✅ Active |
| Cohere | Reranking | ✅ Active |
| OpenAI | Embeddings (fallback) | 🔄 Planned |

## Database Schema

```sql
-- entries_chunks table
CREATE TABLE entries_chunks (
  id UUID PRIMARY KEY,
  entry_id UUID REFERENCES entries(id),
  content TEXT NOT NULL,
  chunk_index INTEGER NOT NULL,
  embedding vector(1024),
  metadata JSONB
);

CREATE INDEX ON entries_chunks USING ivfflat (embedding vector_cosine_ops);
```

## Performance Considerations

- Index uses IVFFlat for approximate nearest neighbor
- Batch embeddings to reduce API calls
- Cache frequent queries (planned)
- Rerank only top-N from vector search

## Future Enhancements

- [ ] Multi-modal search (images, audio)
- [ ] Personal search preferences learning
- [ ] Cross-user search for teams
- [ ] Real-time index updates

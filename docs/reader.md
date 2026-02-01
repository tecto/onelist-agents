# Reader Agent

The Reader Agent processes content, extracts structured memories, and coordinates with LLM providers for intelligent content understanding.

## Status: ✅ Active

## Overview

Reader transforms unstructured content into structured, searchable memories. It's the intelligence layer that understands what users save and how it connects to existing knowledge.

## Architecture

```
Content → Extractors → LLM Processing → Memory Storage → Embedding
              ↓              ↓               ↓
    Parse structure    Generate summary   Create atomic
    Extract entities   Tag suggestion     memories
```

## Components

### 1. Memory Module
- Manages atomic memory units
- Links memories to source entries
- Tracks memory provenance

### 2. Extractors

#### Atomic Memory Extractor
- Breaks content into discrete facts
- Preserves relationships between facts
- Handles various content types

### 3. Generators

#### Tag Suggester
- Analyzes content for relevant tags
- Suggests from existing user tags
- Creates new tags when appropriate
- Respects user's tagging style

### 4. Workers (Oban)

#### ProcessEntryWorker
- Processes new entries async
- Extracts memories
- Generates embeddings
- Updates search index

#### EmbedMemoriesWorker
- Batch embeds new memories
- Handles retry logic
- Rate limits API calls

### 5. Providers

#### Anthropic Provider
- Primary LLM for processing
- Claude 3.5 Sonnet for quality
- Structured output parsing

#### OpenAI Provider
- Fallback provider
- GPT-4 compatibility
- Function calling support

## Configuration

```elixir
# config/config.exs
config :onelist, Onelist.Reader,
  primary_provider: :anthropic,
  model: "claude-sonnet-4-20250514",
  fallback_provider: :openai,
  fallback_model: "gpt-4o",
  max_tokens: 4096,
  temperature: 0.3
```

## Usage

```elixir
# Process an entry
{:ok, memories} = Onelist.Reader.process_entry(entry)

# Generate tag suggestions
{:ok, tags} = Onelist.Reader.suggest_tags(content, existing_tags)

# Extract memories manually
{:ok, memories} = Onelist.Reader.extract_memories(content)
```

## LLM Provider Behaviour

```elixir
@callback complete(prompt, opts) :: {:ok, response} | {:error, reason}
@callback stream(prompt, opts) :: {:ok, stream} | {:error, reason}
@callback count_tokens(text) :: {:ok, count} | {:error, reason}
```

## Memory Schema

```sql
-- memories table
CREATE TABLE memories (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  entry_id UUID REFERENCES entries(id),
  content TEXT NOT NULL,
  type VARCHAR(50),
  confidence FLOAT,
  metadata JSONB,
  embedding vector(1024),
  created_at TIMESTAMP
);
```

## Processing Pipeline

1. **Intake:** Entry created/updated triggers worker
2. **Parse:** Content type detection and preprocessing
3. **Extract:** LLM extracts atomic memories
4. **Tag:** Tag suggestions generated
5. **Embed:** Memories get vector embeddings
6. **Index:** Search index updated
7. **Link:** Cross-references to related memories

## Trusted Memory Mode

For AI users (like Stream), a special mode allows:
- Direct memory creation without source entries
- Higher confidence scores
- Privileged access patterns

## Future Enhancements

- [ ] Multi-modal content processing (images, PDFs)
- [ ] Memory consolidation (merge similar memories)
- [ ] Temporal reasoning (time-aware memories)
- [ ] Memory importance scoring

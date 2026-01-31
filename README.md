# Onelist Agents

AI agents powering Onelist's intelligent features.

## Overview

Onelist uses a multi-agent architecture where specialized AI agents collaborate to provide intelligent note management, semantic search, and conversational interfaces.

## Agents

| Agent | Status | Description |
|-------|--------|-------------|
| [Searcher](docs/searcher.md) | ✅ Active | Semantic search with embeddings and reranking |
| [Reader](docs/reader.md) | ✅ Active | Content processing, memory extraction, and LLM coordination |
| [River](docs/river.md) | ✅ Active | Conversational interface with intent classification |
| [Asset Enrichment](docs/asset-enrichment.md) | 📋 Planned | Automatic metadata enhancement for files and media |

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                          User Interface                          │
└─────────────────────────────┬───────────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────────┐
│                         River Agent                              │
│  (Intent Classification → Entity Extraction → Response Gen)      │
└──────────────┬──────────────────────────────────┬───────────────┘
               │                                  │
┌──────────────▼──────────────┐    ┌──────────────▼──────────────┐
│       Searcher Agent        │    │        Reader Agent         │
│  • Embedding Generation     │    │  • Content Processing       │
│  • Two-Layer Search         │    │  • Memory Extraction        │
│  • Reranking (Cohere)       │    │  • Tag Suggestion           │
│  • Query Reformulation      │    │  • LLM Provider Abstraction │
└──────────────┬──────────────┘    └──────────────┬──────────────┘
               │                                  │
┌──────────────▼──────────────────────────────────▼───────────────┐
│                    PostgreSQL + pgvector                         │
│                (Embeddings, Memories, Entries)                   │
└─────────────────────────────────────────────────────────────────┘
```

## Quick Start

Agents are built into the Onelist Phoenix application. See individual agent docs for configuration.

### Environment Variables

```bash
# Searcher
COHERE_API_KEY=           # For reranking
VOYAGE_API_KEY=           # For embeddings

# Reader
ANTHROPIC_API_KEY=        # Claude for content processing
OPENAI_API_KEY=           # GPT fallback

# River
# Uses Reader's LLM configuration
```

## Development

The agent implementations live in the main [onelist.com](https://github.com/trinsik-labs/onelist.com) repository:

```
lib/onelist/
├── searcher/           # Semantic search agent
│   ├── chunker.ex
│   ├── embedding_job.ex
│   ├── query_reformulator.ex
│   ├── reranker.ex
│   ├── two_layer_search.ex
│   └── providers/
├── reader/             # Content processing agent
│   ├── memory.ex
│   ├── extractors/
│   ├── generators/
│   ├── providers/
│   └── workers/
└── river/              # Conversational agent
    ├── chat/
    │   ├── intent_classifier.ex
    │   ├── entity_extractor.ex
    │   └── response_generator.ex
    ├── entries.ex
    ├── gtd.ex
    └── message.ex
```

## License

MIT License - see [LICENSE](LICENSE) for details.

---

*Part of the [Onelist](https://onelist.my) project by Trinsik Labs*

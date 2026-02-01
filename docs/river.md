# River Agent

The River Agent provides a conversational interface for interacting with Onelist, understanding user intent and generating contextual responses.

## Status: ✅ Active

## Overview

River is Onelist's chat interface—a natural language gateway to your notes, tasks, and memories. It classifies intent, extracts relevant entities, and generates helpful responses using your personal knowledge base.

## Architecture

```
User Message → IntentClassifier → EntityExtractor → Query/Action → ResponseGenerator
                    ↓                   ↓                ↓               ↓
             Determine intent    Extract entities   Search/Create   Format response
             (search/create/     (dates, tags,      (via Searcher/  (natural language)
              update/chat)        references)        Reader)
```

## Components

### 1. Intent Classifier
Determines what the user wants to do:

| Intent | Description | Example |
|--------|-------------|---------|
| `search` | Find information | "What did I note about the meeting?" |
| `create` | Add new entry | "Remember that the deadline is Friday" |
| `update` | Modify existing | "Add a tag to that note" |
| `list` | Show items | "Show my tasks for today" |
| `chat` | General conversation | "What do you think about this idea?" |
| `gtd` | Task management | "What's next on my list?" |

### 2. Entity Extractor
Identifies structured data in natural language:

- **Dates:** "next Tuesday", "in 3 days", "yesterday"
- **Tags:** Implicit and explicit tag references
- **References:** "that note", "the meeting notes", "my todo"
- **Actions:** Verbs indicating desired operations

### 3. Query Module
- Translates intent + entities into database queries
- Coordinates with Searcher for semantic search
- Handles complex multi-step queries

### 4. Response Generator
- Formats results as natural language
- Maintains conversation context
- Handles follow-up questions
- Knows when to ask for clarification

### 5. GTD (Getting Things Done)
- Task prioritization logic
- "What's next?" intelligence
- Context-aware task suggestions
- Due date awareness

## Configuration

```elixir
# config/config.exs
config :onelist, Onelist.River,
  max_context_messages: 10,
  response_model: "claude-sonnet-4-20250514",
  intent_model: "claude-sonnet-4-20250514",
  enable_gtd: true
```

## Usage

```elixir
# Send a message
{:ok, response} = Onelist.River.Chat.send_message(session_id, "Find my notes about project X")

# Start a new session
{:ok, session} = Onelist.River.Chat.create_session(user_id)

# Get GTD suggestion
{:ok, task} = Onelist.River.GTD.next_action(user_id)
```

## Message Flow

```elixir
# 1. User sends message
%Message{content: "What meetings do I have tomorrow?"}

# 2. Intent classification
%{intent: :search, confidence: 0.95}

# 3. Entity extraction
%{
  entities: [
    %{type: :date, value: ~D[2026-02-01], text: "tomorrow"},
    %{type: :tag, value: "meetings", implicit: true}
  ]
}

# 4. Query execution via Searcher
results = Searcher.search(user_id, "meetings", date_filter: tomorrow)

# 5. Response generation
"You have 2 meetings tomorrow:
 - 10am: Team standup
 - 2pm: Project review with Sarah"
```

## Session Management

```sql
-- river_sessions table
CREATE TABLE river_sessions (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  context JSONB,
  last_activity TIMESTAMP,
  created_at TIMESTAMP
);

-- river_messages table
CREATE TABLE river_messages (
  id UUID PRIMARY KEY,
  session_id UUID REFERENCES river_sessions(id),
  role VARCHAR(20),
  content TEXT,
  metadata JSONB,
  created_at TIMESTAMP
);
```

## LiveView Integration

River is exposed via Phoenix LiveView for real-time chat:

```elixir
# lib/onelist_web/live/app/river_live.ex
defmodule OnelistWeb.App.RiverLive do
  use OnelistWeb, :live_view
  
  def handle_event("send_message", %{"content" => content}, socket) do
    # Process through River agent
  end
end
```

## API Endpoint

```elixir
# POST /api/v1/river/message
%{
  "session_id" => "optional-existing-session",
  "message" => "What's on my mind lately?"
}

# Response
%{
  "response" => "Based on your recent notes...",
  "session_id" => "uuid",
  "suggestions" => ["Show more", "Search deeper"]
}
```

## Future Enhancements

- [ ] Voice input/output
- [ ] Multi-turn task completion
- [ ] Proactive suggestions ("You might want to review...")
- [ ] Calendar integration
- [ ] External tool use (web search, calculations)

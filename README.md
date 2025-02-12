# Chat State Actor

This actor manages the state of chat conversations using a directed acyclic graph (DAG) structure. It's designed to support features like conversation branching, persistent storage, and efficient message deduplication.

## Core Concepts

### Message DAG Structure
Chat history is stored as a DAG where:
- Each message is a node in the graph
- Each message has a pointer to its parent message
- Messages are immutable and content-addressed using SHA1 hashes
- Multiple chat threads can share common history

Example:
```
Message1 <- Message2 <- Message3 (Chat A head)
                    <- Message4 (Chat B head)
```

### Data Storage
All data is stored in a flat key-value structure:
- `data/<sha1>.json` - Message objects identified by their content hash
- `data/<title>.json` - Chat objects identified by their title

This design:
- Naturally deduplicates shared message history
- Makes branching conversations trivial
- Allows for easy backup and synchronization
- Enables future features like message linking or forking

### Message Format
```json
{
  "role": "user"|"assistant",
  "content": "message text",
  "parent": "sha1_hash_of_parent_message", // null for root
  "id": "sha1_hash_of_this_message"
}
```

### Chat Format
```json
{
  "title": "Chat Title",
  "head": "sha1_hash_of_latest_message"
}
```

## Key Features

### Persistence
- All messages and chat metadata are automatically persisted to disk
- Messages are immutable and content-addressed
- Chat state can be rebuilt from disk on actor restart

### Branching Support
The DAG structure enables:
- Creating alternate conversation branches
- Maintaining multiple chat heads that share history
- Future features like "what-if" exploration of different responses

### Message Caching
- Recently accessed messages are cached in memory
- Full history can always be reconstructed from disk

## Event Types

### `new_chat`
Create a new chat thread:
```json
{
  "type": "new_chat",
  "title": "Chat Title"
}
```

### `user_message`
Add a user message to current chat:
```json
{
  "type": "user_message",
  "text": "message content"
}
```

### `assistant_response`
Record assistant's response:
```json
{
  "type": "assistant_response",
  "text": "response content"
}
```

### `switch_chat`
Switch to a different chat:
```json
{
  "type": "switch_chat",
  "title": "Chat Title"
}
```

## Why These Design Choices?

### Why a DAG?
1. **Natural Branching**: The DAG structure makes it trivial to create alternate conversation paths from any point in history.
2. **Efficient Storage**: Common history is automatically shared between branches.
3. **Immutable History**: Messages are immutable and content-addressed, making the system more reliable and easier to reason about.
4. **Future Flexibility**: The structure can easily accommodate new features like:
   - Message linking across conversations
   - Parallel exploration of different responses
   - Conversation merging
   - Undo/redo functionality

### Why Flat Storage?
1. **Simplicity**: A flat key-value store is easy to understand and maintain
2. **Deduplication**: Shared messages are automatically stored only once
3. **Easy Backup**: Simple to copy, sync, or backup the data directory
4. **Flexibility**: New types of objects can be added without changing the storage structure

### Why Content Addressing?
1. **Deduplication**: Identical messages are automatically deduplicated
2. **Integrity**: Easy to verify message content hasn't been modified
3. **Caching**: Content-based addressing makes caching more effective
4. **Distribution**: Enables future distributed features

## Future Possibilities

The current design enables several future enhancements:
1. Message annotations or metadata
2. Alternative LLM responses at any point
3. Conversation merging
4. Distributed chat storage
5. Advanced chat management features
6. Message searching and linking

## Interaction with Other Components

### Browser UI
- Receives formatted messages for display
- Will need updates to support branching visualization

### LLM Gateway
- Receives full message history for context
- Could be extended to support generating alternative responses
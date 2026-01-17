## 10. Local Memory OS Architecture (Letta Integration)

For persistent local inference on M5 Max hardware, the system requires explicit context window management to prevent crashes during long-running sessions.

### 10.1 The "Memory OS" Pattern

The local agent's context window (32k-60k tokens depending on quantization) is sliced into three protected sectors:

| Sector | Size | Purpose | Persistence |
|--------|------|---------|-------------|
| **System Instructions** | ~1k | Identity & Prime Directives | Permanent (Cached) |
| **The "Notebook"** | ~2k | Writable memory managed by agent | Evolving (Rewritten by Agent) |
| **The Sliding Window** | ~29-57k | Active conversation / Code View | Transient (Flushed regularly) |

### 10.2 Context Pressure Triggers

When the sliding window approaches capacity (~95%), the system:

1. **Pauses** user interaction
2. **Prompts** the agent: "Update your Notebook with critical facts from recent conversation, and commit data to Vector DB"
3. **Agent rewrites** the Notebook sector with summary
4. **Script flushes** the Sliding Window
5. **Script injects** the updated Notebook
6. **Resumes** with fresh short-term memory but intact summary

### 10.3 Letta Framework Integration

**Letta** (formerly MemGPT) is the reference implementation for this "Memory OS" pattern:

- Handles context pressure detection automatically
- Manages the Notebook rewrite protocol
- Integrates with local `llama-server` backends
- Provides archival memory via ChromaDB for long-term storage

**Implementation Note**: When M5 Max arrives, Letta will manage the Command R+ inference engine, pointing to a local `llama-server` API hosting the model. This decouples brain (inference) from mind (memory logic), preventing crashes if one component hangs.

### 10.4 Dual Memory Architecture

The local system uses two complementary memory stores:

- **Hippocampus (Letta + ChromaDB)**: Active retrieval for current tasks. Local vector store for "What do I need to know right now?"
- **Cortex (Supabase)**: Evolutionary logs for TIES merging. Structured storage for "What should I integrate into my weights next week?"

See Section 2 for Supabase schema details. ChromaDB runs locally and stores archival facts outside the context window, queried on-demand by Letta.

---

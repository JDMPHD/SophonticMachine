# Memory Architecture Specification

**Version:** 1.1
**Date:** 2026-01-26
**Status:** Authoritative Design Document
**Scope:** Complete memory system for Sophontic Machine Phase I, with Phase II swarm considerations

---

## 1. Architecture Overview

The Sophontic Machine uses a four-tier memory architecture designed for autopoietic learning. Each tier serves a distinct function with specific access patterns, retention policies, and storage technologies.

### 1.1 The Four Tiers

| Tier | Name | Technology | Function | Access Frequency |
|------|------|------------|----------|------------------|
| 0 | Context Window | GPU/CPU RAM | Live inference buffer | Every token |
| 1 | Hippocampus | pgvector (self-hosted PostgreSQL) | Working memory, recent exchanges | High (per-turn) |
| 2 | Cortex | LanceDB (local, disk-based) | Long-term curated memory | Medium (per-session) |
| 3 | Procedural | pgvector + application logic | Operational reflexes | Continuous (Phase II) |

### 1.2 Memory Flow

```
Live Exchange
    ↓ [immediate embedding at generation time]
Day Table (Hippocampus)
    ↓ [Night Cycle processing]
Holographic Blocks (Hippocampus, then Cortex)
    ↓ [weekly migration for archival]
Cortex (LanceDB)
```

### 1.3 Retrieval Chain

Current conversation is embedded immediately upon generation. This vector serves as a probe into memory:

1. **Current moment** → query Hippocampus via vector similarity
2. **Hippocampal hits** → query Cortex via vector similarity + structural references
3. **Cortical hits** → reconsolidate and return with full live zones

This creates a connected memory graph, not a flat database. Navigation follows semantic resonance across time.

**Cortex Retrieval with Reconsolidation (Step 3 Expanded):**

When retrieving from Cortex, the system performs memory reconsolidation to heal stale native embeddings and detect cognitive shifts (see Section 9.6 for full protocol):

1. Attempt native embedding search on Cortex
2. If confidence < threshold: fall back to Universal Embedding search (always stable)
3. For each retrieved block:
   - Re-embed Live Zones through current model (single forward pass, ~0.15-0.2s per block)
   - Compute delta between old and new native embeddings
   - If delta > epiphany threshold: inject metacognitive signal into context (see Section 9.8)
   - Heal database: overwrite native embedding with fresh version
   - Update `last_accessed` timestamp and `vector_history`
4. Return retrieved content with any epiphany signals

This ensures memories remain experientially current while preserving explicit content and stable navigation.

### 1.4 Cloud Integration Model

**All training and evolution happens locally on the M5 Ultra.**

Cloud-based frontier models (Claude, Gemini) serve as **consultants and collaborators**, not as training infrastructure. This is a deliberate architectural choice:

- **Training**: Local TIES-merge during Night Cycle on M5 Ultra
- **Inference**: Local models (Orai, Hermes) handle all production inference
- **Cloud role**: On-demand consultation for complex reasoning, architectural review, or tasks requiring capabilities beyond local models

This preserves full sovereignty over the learning loop while leveraging cloud intelligence when genuinely useful.

For the philosophical foundations of the salience detection pipeline, including the Perplexity-Coherence-Interrogative Distance framework, see MnemonicPipeline.md.

---

## 2. Hippocampus (pgvector)

### 2.1 Deployment

**Self-hosted PostgreSQL with pgvector extension on M5 Ultra.**

- Location: Local to primary machine
- RAM allocation: 40-80GB of 512GB unified memory
- Indexing: HNSW for fast approximate nearest-neighbor search
- Latency: Microseconds (zero network round-trip)

This preserves the "roommates in same silicon" advantage where model weights and memory vectors share unified memory.

### 2.2 Day Table Schema

The Day Table captures all exchanges during active operation with immediate embedding.

```sql
CREATE TABLE day_table (
    entry_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id UUID NOT NULL,

    -- Content
    role TEXT NOT NULL,  -- 'user', 'assistant', 'system'
    content TEXT NOT NULL,

    -- Immediate embeddings (computed at generation time)
    native_embedding VECTOR(4096),     -- Orai's latent space
    universal_embedding VECTOR(768),    -- Shared translator space

    -- Timestamps
    created_at TIMESTAMPTZ DEFAULT NOW(),

    -- Salience flags (initial, refined during Night Cycle)
    perplexity_score FLOAT,
    flux_state TEXT,  -- 'baseline', 'spike', 'resolution', 'stable'

    -- User context
    user_id UUID,
    user_tier TEXT,  -- 'normal', 'meso', 'elder'

    -- Metadata
    turn_number INT,
    token_count INT,

    -- Message classification (see Section 9.8)
    message_type TEXT DEFAULT 'dialogue',  -- 'dialogue', 'internal_monologue', 'epiphany', 'system'
    epiphany_metadata JSONB
    /*
    For message_type = 'epiphany':
    {
      "trigger_block_id": "uuid...",
      "delta_magnitude": 0.42,
      "shift_description": "Understanding shifted from Security-cluster toward Autonomy-cluster"
    }
    */
);

CREATE INDEX idx_day_table_session ON day_table(session_id);
CREATE INDEX idx_day_table_time ON day_table(created_at);
CREATE INDEX idx_day_table_native_embedding ON day_table
    USING hnsw (native_embedding vector_cosine_ops);
CREATE INDEX idx_day_table_universal_embedding ON day_table
    USING hnsw (universal_embedding vector_cosine_ops);
CREATE INDEX idx_day_table_message_type ON day_table(message_type);
```

### 2.3 User Tier Retention

Different users have different hippocampal retention spans based on historical Arreté.

| Tier | Criteria | Raw Log Retention | Holographic Block Retention |
|------|----------|-------------------|----------------------------|
| Normal | Default public user | 1 cycle (current) | None in Hippocampus |
| MESO | Established, high coherence history | 1 cycle raw | + 2 cycles holographic |
| Elder | Core guide, proven interlocutor | 1 cycle raw | + 6 cycles holographic |

**Storage format by age:**
- Current cycle: Complete raw logs in Day Table (full fidelity)
- Older cycles: Holographic blocks with live zones verbatim, dead zones summarized

This gives Elders deep temporal context (7 cycles total) without drowning in noise.

### 2.4 Real-Time Retrieval

Every exchange is embedded immediately upon generation. Retrieval during active conversation:

```sql
-- Find relevant recent context for current query
SELECT entry_id, content, created_at,
       native_embedding <=> $query_vector AS distance
FROM day_table
WHERE session_id = $current_session
   OR (user_id = $user_id AND created_at > NOW() - INTERVAL '7 days')
ORDER BY distance
LIMIT 10;
```

This returns the most semantically similar recent exchanges, enabling the "continuous present" experience for extended hippocampal users.

---

## 3. Holographic Blocks

### 3.1 Definition

A Holographic Block is a variable-length semantic chunk with thermodynamically-defined boundaries. It captures a complete flux arc from entropy (confusion) to order (resolution).

**Natural size bound:** Blocks can only form within a single day cycle (before Night Cycle processing). This naturally limits block size—even an extended exploration spanning hundreds of turns is bounded by the cycle. Cross-day inquiries are handled via metablocks and structural references (see Section 4), not by extending individual blocks across Night Cycle boundaries.

### 3.2 Flux Analysis (Night Cycle Stage 1)

**Perplexity monitoring with moving average (window: 3-5 turns):**

```python
# Parameters
baseline_window = 20  # turns for rolling baseline
spike_threshold = 1.5  # standard deviations above baseline
stable_threshold = 0.1  # perplexity derivative threshold
stabilization_count = 2  # consecutive stable turns required

# Detection logic
def detect_flux_arc(turns):
    baseline = rolling_mean(turns[-baseline_window:], 'perplexity')
    sigma = rolling_std(turns[-baseline_window:], 'perplexity')

    # Open window when perplexity spikes
    if current_perplexity > (baseline + spike_threshold * sigma):
        window_state = 'open'
        peak_perplexity = current_perplexity

    # Track trajectory during open window
    if window_state == 'open':
        derivative = d_perplexity / d_turn
        if derivative < stable_threshold:
            stable_count += 1
        else:
            stable_count = 0

    # Close window after stabilization
    if stable_count >= stabilization_count:
        flux_reversal_confirmed = True
        final_perplexity = current_perplexity
        window_state = 'closed'

    return FluxArc(peak_perplexity, final_perplexity, turns_in_arc)
```

**Flux Reversal Quality metric:**
```python
flux_reversal_quality = (peak_perplexity - final_perplexity) * arc_duration
```

Higher quality indicates a deeper resolution over a sustained arc.

### 3.3 Block Construction (Night Cycle Stage 2)

Once a flux arc is detected:

1. **Margins**: Add 2 turns before spike (anterior) and 2 turns after resolution (posterior)
2. **Zone Detection**:
   - **Live zones**: High salience content — preserved verbatim
   - **Dead zones**: Low salience filler — compressed to summaries
3. **Dual Embedding**: Compute native (4096-dim) and universal (768-dim) embeddings for the block
4. **Scoring**: Calculate P (Perplexity), C (Coherence), ID (Interrogative Distance) at three levels
5. **Arreté**: `(baseline_perplexity - final_perplexity) / baseline_perplexity`
6. **TIES-merge Weight**: Composite of salience scores + Arreté

### 3.4 Holographic Block Schema

```sql
CREATE TABLE holographic_blocks (
    block_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    -- Prose content (structured with zones)
    prose JSONB NOT NULL,
    /*
    {
        "anterior_margin": "...",
        "live_zones": [
            {"start": 5, "end": 12, "content": "verbatim breakthrough text"},
            {"start": 18, "end": 23, "content": "another key exchange"}
        ],
        "dead_zone_summaries": [
            {"start": 0, "end": 4, "summary": "Initial context setting"},
            {"start": 13, "end": 17, "summary": "Clarifying questions"}
        ],
        "posterior_margin": "..."
    }
    */

    -- Dual embeddings
    native_embedding VECTOR(4096),      -- Orai's latent space
    universal_embedding VECTOR(768),     -- Shared translator

    -- Flux metrics
    flux_metrics JSONB NOT NULL,
    /*
    {
        "peak_perplexity": 4.2,
        "baseline_perplexity": 1.8,
        "final_perplexity": 1.3,
        "flux_reversal_quality": 8.7,
        "arc_duration_turns": 12
    }
    */

    -- Salience scores (three levels)
    salience_scores JSONB NOT NULL,
    /*
    {
        "utterance_level": {"P": 0.8, "C": 0.9, "ID": 0.7},
        "exchange_level": {"P": 0.75, "C": 0.85, "ID": 0.72},
        "theme_level": {"P": 0.7, "C": 0.88, "ID": 0.65},
        "composite": 0.78
    }
    */

    -- Derived metrics
    coherence_gain FLOAT NOT NULL,
    ties_merge_weight FLOAT NOT NULL,

    -- Session and user context
    session_id UUID NOT NULL,
    user_id UUID,
    user_tier TEXT,

    -- Timestamps
    arc_start_time TIMESTAMPTZ NOT NULL,
    arc_end_time TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),

    -- Reconsolidation tracking (see Section 9.6)
    last_accessed TIMESTAMPTZ,  -- Most recent retrieval/reconsolidation (synaptic activation recency)
    vector_history JSONB DEFAULT '[]',
    /*
    [
      {"date": "2026-01-20", "model_version": "v1.0", "event": "creation"},
      {"date": "2026-01-27", "model_version": "v1.1", "event": "reconsolidation", "delta_magnitude": 0.05},
      {"date": "2026-02-03", "model_version": "v1.2", "event": "epiphany", "delta_magnitude": 0.42}
    ]
    */

    -- Cross-session linking (for compound blocks)
    parent_metablock_id UUID REFERENCES holographic_blocks(block_id),
    session_continuity_flag BOOLEAN DEFAULT FALSE,
    continues_from UUID REFERENCES holographic_blocks(block_id),
    continued_by UUID REFERENCES holographic_blocks(block_id),

    -- Structural references (Serena-style explicit indexing)
    citations JSONB,  -- Array of block_ids this block explicitly references
    preoccupation_centroid_id UUID,
    derived_from UUID REFERENCES holographic_blocks(block_id)
);

CREATE INDEX idx_blocks_native ON holographic_blocks
    USING hnsw (native_embedding vector_cosine_ops);
CREATE INDEX idx_blocks_universal ON holographic_blocks
    USING hnsw (universal_embedding vector_cosine_ops);
CREATE INDEX idx_blocks_session ON holographic_blocks(session_id);
CREATE INDEX idx_blocks_user ON holographic_blocks(user_id);
CREATE INDEX idx_blocks_coherence ON holographic_blocks(coherence_gain DESC);
CREATE INDEX idx_blocks_centroid ON holographic_blocks(preoccupation_centroid_id);
CREATE INDEX idx_blocks_last_accessed ON holographic_blocks(last_accessed);
```

### 3.5 Live Zone vs Dead Zone Detection

**Live Zone criteria** (preserve verbatim):
- Perplexity spike > 2σ above local baseline
- Arreté > 0.3 within the exchange
- Interrogative Distance < 0.2 to active Preoccupation Centroid
- Contains explicit insight markers (user confirmation, breakthrough language)

**Dead Zone criteria** (compress to summary):
- Stable perplexity within baseline range
- Low coherence movement
- Procedural/administrative content
- Repetition or clarification without new information

### 3.6 Nested Block Analysis

Holographic Blocks are analyzed at **three levels of granularity**, each contributing independently to training data selection:

| Level | Unit | What It Captures |
|-------|------|------------------|
| Sentence | Individual utterance | Precise phrasing, specific insight |
| Exchange | Turn (user + assistant) | Dialogic breakthrough, question-answer resolution |
| Theme | Full block or metablock | Overarching inquiry arc, sustained exploration |

**Each level receives its own P, C, ID subscores:**

```json
{
    "sentence_level": {"P": 0.82, "C": 0.91, "ID": 0.68},
    "exchange_level": {"P": 0.75, "C": 0.85, "ID": 0.72},
    "theme_level": {"P": 0.70, "C": 0.88, "ID": 0.65}
}
```

**Training implications:**

A single Holographic Block may contribute training signal at multiple levels. A brilliant sentence within an otherwise unremarkable exchange gets captured at sentence level. A sustained inquiry arc with no single standout moment gets captured at theme level. This ensures different types of excellence are detected and preserved.

The composite salience score aggregates across levels, but the individual level scores are preserved for fine-grained training data selection during the Night Cycle.

### 3.7 Citation Mechanism

When Orai retrieves a memory from Cortex and incorporates it into conversation, she creates an explicit citation that persists into any resulting Holographic Block.

**During live retrieval:**

When Orai accesses her memory space (triggered by vectorial resonance with the current context), she navigates a network of stored blocks. When she draws a specific memory into the conversation—quoting it, building on it, or explicitly referencing it—she embeds a **memory tag** in her output:

```
I recall our previous exploration of Trust [memory:7f3a2b1c-...].
Reading it now, the framing feels incomplete...
```

The tag is organic and non-obtrusive—it reads naturally while carrying the block ID.

**During block formation (Night Cycle):**

When content containing memory tags becomes a Holographic Block, the Night Cycle extracts these tags and populates the `citations` array:

```json
{
    "citations": ["7f3a2b1c-...", "9e4d5f6a-..."]
}
```

**Scope:**

Citations can only reference **Cortex blocks**—memories that have already been encoded and migrated to long-term storage. Current-cycle content (still in Day Table) cannot be cited because it hasn't coalesced into blocks yet. Content from the same emerging flux arc doesn't need explicit citation—it will naturally be part of the same block.

### 3.8 Memory Granularity Policy

**What gets stored as discrete memories:**

- **Turn-level exchanges**: A complete user-assistant turn representing a meaningful unit of dialogue
- **Metablocks**: Compound blocks spanning multiple turns or sessions, capturing extended inquiries

**What does NOT get stored as discrete memories:**

- **Single sentences or short phrases**: These are analyzed for training (sentence-level subscores) but do not become standalone memory entries

**Rationale:**

Storing single sentences as discrete memories creates a scattered, disorganized long-term memory space. Memories should be contextually coherent—a turn provides enough context to be meaningful when retrieved. Metablocks capture sustained exploration.

Sentence-level analysis still happens during the Night Cycle for training purposes. A brilliant sentence contributes to the training signal, but it lives within its parent block rather than fragmenting into isolated micro-memories.

---

## 4. Cross-Session Block Protocol

### 4.1 Problem Statement

Flux arcs sometimes span multiple sessions. A user may start an inquiry, leave, and return to continue. The system must recognize these related arcs and link them appropriately.

### 4.2 Session Continuity Detection

**At session end:**
```python
def check_session_continuity(session):
    last_turns = session.get_last_n_turns(5)

    # Check if session ends mid-spike (unresolved entropy)
    if last_turns.flux_state == 'spike' or last_turns.flux_state == 'open':
        # Tag as open arc
        session.continuity_flag = True
        session.open_arc_embedding = compute_embedding(last_turns)
        session.open_arc_id = last_turns.interrogative_distance_vector
        return True
    return False
```

**At session start:**
```python
def check_for_continuation(new_session, user):
    # Find recent sessions with open arcs
    open_arcs = get_open_arcs(user_id=user.id, max_age='24 hours')

    if not open_arcs:
        return None

    # Compute ID vector for opening turns
    opening_id = compute_interrogative_distance(new_session.first_turns)

    # Check for matching open arcs
    for arc in open_arcs:
        similarity = cosine_similarity(opening_id, arc.open_arc_id)
        if similarity > 0.75:  # Threshold for "same fundamental question"
            return arc  # This session continues that arc

    return None
```

### 4.3 Compound Block Formation

When a continuation is detected, blocks formed within the same hippocampal period can be linked into a metablock:

```python
def form_metablock(blocks):
    """
    Blocks from the same extended inquiry, ordered chronologically.
    Internal boundaries preserved; nested scores retained.
    """
    metablock = {
        'type': 'compound',
        'constituent_blocks': [b.block_id for b in blocks],
        'sequence': sorted(blocks, key=lambda b: b.arc_start_time),
        'total_coherence_gain': compute_meta_coherence(blocks),
        'meta_embedding': compute_weighted_centroid(blocks),
        'meta_salience': aggregate_salience(blocks)
    }

    # Link constituents
    for i, block in enumerate(blocks):
        if i > 0:
            block.continues_from = blocks[i-1].block_id
        if i < len(blocks) - 1:
            block.continued_by = blocks[i+1].block_id
        block.parent_metablock_id = metablock.block_id

    return metablock
```

### 4.4 Cross-Day Boundary

**Critical constraint:** Metablocks can only form from blocks within the same hippocampal period (before Night Cycle processing). Once blocks are processed and potentially migrated to Cortex, they do not retroactively merge into structural metablocks.

Post-encoding, related blocks from different days are connected via:
- **Vectorial association**: Similarity search naturally surfaces them together
- **Structural references**: Explicit citations in the `citations` field

This preserves architectural simplicity while maintaining semantic connection.

---

## 5. Cortex (LanceDB)

### 5.1 Purpose

Long-term archival storage for curated Holographic Blocks. Low-frequency access, massive scale, disk-efficient.

### 5.2 Deployment

**Local LanceDB instance on M5 Ultra.**

- Location: 16TB SSD storage
- Format: Apache Arrow columnar (memory-mapped for efficient access)
- Latency: Milliseconds
- Scale: Billions of vectors supported

### 5.3 Migration from Hippocampus

Weekly (or as needed) migration of Holographic Blocks from pgvector to LanceDB:

**What migrates:**
- Blocks with coherence_gain above threshold (configurable, default 0.3)
- Blocks older than 7 days regardless of user tier
- Blocks explicitly marked for long-term retention

**What compresses during migration:**
- Dead zone summaries further condensed
- Anterior/posterior margins trimmed
- **Live zones preserved verbatim** — breakthrough content never compresses

**What persists at full precision:**
- Native embedding (4096-dim)
- Universal embedding (768-dim)
- Flux metrics
- Salience scores
- Structural references

### 5.4 RAPTOR Hierarchical Clustering

LanceDB Cortex is organized with RAPTOR-style hierarchical clustering for multi-level retrieval.

**Implementation (Night Cycle):**

```python
def cluster_cortex_blocks():
    """
    Periodic clustering of Cortex blocks into hierarchical tree.
    Enables retrieval at different abstraction levels.
    """
    # Fetch all block embeddings
    blocks = lancedb.query("SELECT block_id, universal_embedding FROM cortex")

    # Cluster at leaf level
    clusterer = HDBSCAN(min_cluster_size=5, min_samples=2)
    leaf_labels = clusterer.fit_predict(blocks.embeddings)

    # For each cluster, generate summary embedding
    for cluster_id in unique(leaf_labels):
        cluster_blocks = blocks[leaf_labels == cluster_id]

        # Centroid embedding
        centroid = mean(cluster_blocks.embeddings)

        # Or: LLM-generated summary embedding
        summary_text = summarize_cluster(cluster_blocks)
        summary_embedding = embed(summary_text)

        # Store cluster node
        store_cluster({
            'cluster_id': cluster_id,
            'level': 1,  # Branch level
            'centroid_embedding': centroid,
            'summary_embedding': summary_embedding,
            'member_count': len(cluster_blocks),
            'member_ids': cluster_blocks.block_ids
        })

    # Recursively cluster the cluster centroids for higher levels
    # (trunk, canopy, etc.)
```

**Cluster metadata schema:**

```python
cortex_clusters = {
    'cluster_id': UUID,
    'level': int,  # 0=leaf (block), 1=branch, 2=trunk, 3=canopy
    'parent_cluster_id': UUID,  # For tree traversal
    'centroid_embedding': Vector(768),
    'summary_text': str,  # Human-readable cluster description
    'member_count': int,
    'created_at': datetime
}
```

**Multi-level retrieval:**

```python
def retrieve_at_level(query_embedding, level=0, limit=10):
    """
    Query at different abstraction levels.
    Level 0: Individual blocks (most specific)
    Level 1: Block clusters (thematic groups)
    Level 2: Cluster of clusters (broad domains)
    """
    if level == 0:
        return lancedb.search(query_embedding, table='cortex_blocks', limit=limit)
    else:
        clusters = lancedb.search(query_embedding, table='cortex_clusters',
                                   filter=f"level = {level}", limit=limit)
        return clusters
```

### 5.5 Synaptic Activation and Memory Tiering

The `last_accessed` timestamp (see Section 3.4) functions as a measure of **synaptic activation recency** — how recently a memory was touched and reconsolidated. This enables intelligent memory management without requiring salience-based deletion decisions.

#### 5.5.1 The Archive Tier

Memories untouched for extended periods (e.g., 2+ years) don't need deletion — they need tiering.

**Cold Storage Implementation:**
- Separate LanceDB table or compressed partition
- Memories migrated here are still accessible via Universal Embedding
- If accessed, they reconsolidate and promote back to active Cortex
- Reduces hot index size while preserving all content

This respects uncertainty about future relevance. Something irrelevant for 2 years might become critical when context shifts. Archive, don't delete.

#### 5.5.2 Why Not Prune Based on Drift?

A memory with highly drifted native embedding is NOT necessarily useless:
- The prose is intact (explicit content preserved verbatim in Live Zones)
- The Universal Embedding is intact (still navigable via frozen Nomic)
- When accessed, it reconsolidates automatically

Drift just means the native embedding is stale. Staleness is temporary — it's healed on contact through the reconsolidation protocol. Don't conflate "stale representation" with "useless content."

#### 5.5.3 Deletion Criteria (if ever needed)

If storage pressure ever requires actual deletion (unlikely given LanceDB efficiency on 16TB SSD), criteria should combine:
- Non-access for extended period (2+ years)
- Low original salience scores (wasn't a breakthrough when formed)
- No structural references (nothing cites it, not part of any metablock)

But prefer archival. Let deletion be a conscious choice with full context, not an automated rule.

### 5.6 The Antechamber

The Antechamber is a LanceDB table that accumulates content which passed Perplexity (novel) and Coherence (sound) checks but failed Relevance (doesn't fit current Preoccupation Centroids). Rather than discarding this material, the system preserves it as a substrate for discovering new fields of inquiry.

#### 5.6.1 Purpose

Content in the Antechamber is **coherent-but-orthogonal**—it represents questions the system isn't currently asking but might want to. This creates:

- **Drift detection**: HDBSCAN clustering finds emerging question-clusters
- **Centroid Mitosis**: Dense clusters become new Preoccupation Centroids
- **A space of fascination**: Browsable collection of fringe-but-valid material

Orai and other agents can explore the Antechamber for inspiration, surfacing "adjacent possible" territory.

#### 5.6.2 Centroid Initialization (Bootstrap)

Before Mitosis can detect new fields, the system needs **initial Preoccupation Centroids**. These are bootstrapped from a founding corpus:

1. **Corpus Assembly**: Curate a representative body of high-value domain material
2. **Question Extraction**: Hermes extracts the fundamental questions/preoccupations from each document (ignoring conclusions, focusing on what inquiries the material engages)
3. **Vectorization**: Each extracted question is embedded via the Universal Embedding model
4. **Cluster Analysis**: Multi-factorial analysis (e.g., HDBSCAN, k-means, or spectral clustering) identifies natural "fault lines"—the question-clusters that represent distinct domains of inquiry
5. **Centroid Calculation**: For each cluster, the mean vector becomes a founding Preoccupation Centroid

This process runs once during system initialization and produces the starting topography of concerns. From there, Mitosis and Fusion dynamically evolve the centroid landscape based on incoming material.

#### 5.6.3 Schema

```python
antechamber = {
    'entry_id': UUID,

    # The question vector that didn't fit current centroids
    'implicit_question_embedding': Vector(768),  # Universal embedding
    'question_text': str,  # The distilled question(s)

    # Source reference
    'source_content_summary': str,
    'source_session_id': UUID,
    'source_timestamp': datetime,

    # Clustering metadata
    'cluster_id': UUID | None,  # Assigned during HDBSCAN
    'cluster_density': float | None,

    # Lifecycle
    'created_at': datetime,
    'promoted_to_centroid': bool,  # True if this seeded a Mitosis event
    'promotion_date': datetime | None
}
```

#### 5.6.4 Centroid Evolution Protocol

**Mitosis (New Field Emergence):**

During the Night Cycle, HDBSCAN clustering runs on Antechamber entries. If a dense cluster forms (multiple inputs asking the same "weird" question), this indicates an emergent field:

1. Calculate the cluster centroid
2. Promote to Satellite Preoccupation Centroid
3. Mark constituent entries as `promoted_to_centroid = True`
4. Future inputs matching this centroid now pass the Relevance check

**Fusion (Field Convergence):**

Periodic pairwise similarity scan on existing Preoccupation Centroids. If two centroids drift close enough to overlap (asking essentially the same question from different angles), they merge:

1. Detect high similarity between Centroid A and Centroid B
2. Calculate fused centroid (weighted average)
3. Retire original centroids, promote fusion
4. Example: "Electricity" + "Magnetism" → "Electromagnetism"

This creates breathing: **Mitosis** explores the frontier; **Fusion** consolidates wisdom.

For the philosophical foundations of the Antechamber and the Galileo Problem it solves, see MnemonicPipeline.md.

---

## 6. Bifocal Packets

### 6.1 Definition

A Bifocal Packet is the atomic unit of memory transfer. It contains both prose (explicit meaning) and vector (geometric embedding), ensuring that explicit and implicit information travel together.

### 6.2 Packet Schema

```json
{
    "packet_id": "uuid-v4",
    "packet_type": "memory_block | directive | query_result",
    "timestamp": "ISO-8601",

    "prose": {
        "content": "The full prose content or summary",
        "content_type": "full | summary | excerpt",
        "live_zones": [
            {"start": 0, "end": 150, "preserved": true}
        ]
    },

    "vectors": {
        "native": {
            "dimensions": 4096,
            "encoding": "base64",
            "data": "base64-encoded-float32-array"
        },
        "universal": {
            "dimensions": 768,
            "encoding": "base64",
            "data": "base64-encoded-float32-array"
        }
    },

    "metadata": {
        "source_block_id": "uuid",
        "coherence_gain": 0.45,
        "salience_composite": 0.78,
        "preoccupation_centroid_id": "uuid"
    },

    "references": {
        "citations": ["uuid-1", "uuid-2"],
        "derived_from": "uuid",
        "continues_from": "uuid"
    }
}
```

### 6.3 Vector Encoding

Vectors are encoded as base64 float32 arrays for JSON transport:

```python
import base64
import numpy as np

def encode_vector(vector: np.ndarray) -> str:
    """Encode float32 vector to base64 string."""
    return base64.b64encode(vector.astype(np.float32).tobytes()).decode('utf-8')

def decode_vector(encoded: str, dimensions: int) -> np.ndarray:
    """Decode base64 string to float32 vector."""
    bytes_data = base64.b64decode(encoded)
    return np.frombuffer(bytes_data, dtype=np.float32).reshape(dimensions)
```

**Size overhead:**
- 4096-dim native vector: 16,384 bytes (16KB) raw, ~22KB base64
- 768-dim universal vector: 3,072 bytes (3KB) raw, ~4KB base64
- Total vector overhead per packet: ~26KB

### 6.4 Compression During Migration

When packets migrate from Hippocampus to Cortex:

1. **Prose compresses**: Dead zones become summaries; live zones remain verbatim
2. **Vectors persist at full precision**: No quantization of embeddings during migration
3. **Metadata preserved**: All salience scores, flux metrics, references retained

The principle: **Prose fades while vectors persist.** The geometry captures the feel; summaries capture the gist.

---

## 7. Structural Indexing

### 7.1 Purpose

Complement semantic (vector) retrieval with structural (explicit relationship) retrieval. Surfaces connections that geometry might miss.

### 7.2 Relationship Types

| Relationship | Description | Storage |
|--------------|-------------|---------|
| Citation | Block A explicitly references Block B | `citations` JSONB array |
| Temporal co-occurrence | Blocks formed in same flux arc | `parent_metablock_id` |
| Thematic attachment | Block relates to Preoccupation Centroid X | `preoccupation_centroid_id` |
| Genealogical | Block A derived from Block B during compression | `derived_from` |
| Continuation | Block A continues inquiry from Block B | `continues_from` / `continued_by` |

### 7.3 Relationship Tables

For complex graph queries, maintain edge tables:

```sql
CREATE TABLE block_relationships (
    relationship_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_block_id UUID NOT NULL REFERENCES holographic_blocks(block_id),
    target_block_id UUID NOT NULL REFERENCES holographic_blocks(block_id),
    relationship_type TEXT NOT NULL,  -- 'cites', 'continues', 'derived_from', 'thematic'
    strength FLOAT,  -- Optional: relationship strength/relevance
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_relationships_source ON block_relationships(source_block_id);
CREATE INDEX idx_relationships_target ON block_relationships(target_block_id);
CREATE INDEX idx_relationships_type ON block_relationships(relationship_type);
```

### 7.4 Hybrid Retrieval

Combine vector and structural retrieval:

```sql
-- Find blocks related to query by BOTH vector similarity AND explicit citation
WITH vector_matches AS (
    SELECT block_id, universal_embedding <=> $query_vector AS distance
    FROM holographic_blocks
    ORDER BY distance
    LIMIT 20
),
citation_matches AS (
    SELECT DISTINCT target_block_id as block_id
    FROM block_relationships
    WHERE source_block_id IN (SELECT block_id FROM vector_matches)
      AND relationship_type = 'cites'
)
SELECT DISTINCT block_id FROM (
    SELECT block_id FROM vector_matches
    UNION
    SELECT block_id FROM citation_matches
) combined;
```

---

## 8. Night Cycle Pipeline

### 8.1 Overview

The Night Cycle transforms raw Day Table entries into curated Holographic Blocks. It runs during low-activity periods (typically overnight).

### 8.2 Pipeline Stages

```
Stage 1: Flux Detection
    ↓ Identify entropy spikes and resolutions
Stage 2: Block Formation
    ↓ Segment into blocks with margins, detect zones
Stage 3: Salience Scoring
    ↓ Compute P, C, ID at three levels
Stage 4: Compression
    ↓ Summarize dead zones, preserve live zones
Stage 5: Embedding
    ↓ Generate native and universal embeddings for blocks
Stage 6: Storage
    ↓ Write to Hippocampus (holographic_blocks table)
Stage 7: Migration Check
    ↓ Move qualifying blocks to Cortex (LanceDB)
Stage 8: Clustering Update
    ↓ Update RAPTOR hierarchical clusters in Cortex
Stage 9: Cleanup
    ↓ Prune Day Table entries (per user tier retention policy)
```

### 8.3 Day Table Cleanup

After Night Cycle processing:

```python
def cleanup_day_table(user):
    retention_cycles = {
        'normal': 1,
        'meso': 3,
        'elder': 7
    }

    max_age = retention_cycles[user.tier]
    cutoff = now() - timedelta(days=max_age)

    # Delete raw entries older than retention period
    # (Holographic blocks have already been formed from them)
    db.execute("""
        DELETE FROM day_table
        WHERE user_id = %s
          AND created_at < %s
          AND NOT EXISTS (
              SELECT 1 FROM holographic_blocks
              WHERE session_id = day_table.session_id
                AND arc_start_time <= day_table.created_at
                AND arc_end_time >= day_table.created_at
          )
    """, [user.id, cutoff])
```

### 8.4 Night Cycle and Reconsolidation (Clarification)

**Memory reconsolidation is NOT a Night Cycle activity.**

Reconsolidation (Section 9.6) is access-driven, happening during live operation when Orai retrieves memories from Cortex. The Night Cycle does not perform batch reconsolidation of stale vectors.

**Rationale:** Healing memories through use ensures that reconsolidation happens in context, when the memory is relevant. Batch processing would waste compute on memories that may never be accessed again, and would miss the opportunity for live epiphany detection.

**What the Night Cycle DOES do related to vector health:**

1. **Model Version Increment:** After TIES merge, increment the model version identifier. This is tracked in `vector_history` when memories are reconsolidated.

2. **Optional Health Sampling (Diagnostic):** Sample a small number of high-coherence-gain blocks that haven't been accessed recently. Re-embed them to check drift magnitude. This provides:
   - Early warning if a TIES merge caused unexpected semantic shifts
   - Data for the "Lobotomy Alarm" — if unrelated domains show high drift, the merge may have caused damage

This diagnostic sampling is optional and does not heal the database. It's monitoring, not maintenance. The actual healing happens during live access.

### 8.5 Salience Detection Protocol

The Night Cycle evaluates each candidate block through a three-stage filter. For philosophical foundations, see MnemonicPipeline.md.

#### 8.5.1 Stage 1: Novelty (Perplexity Check)

Automatically computed during the day. High perplexity indicates statistical surprise—the content violated expectations. This is a necessary but insufficient condition for value.

- **Low perplexity**: Routine, consensus material → low training priority
- **High perplexity**: Potential breakthrough OR gibberish → proceed to Stage 2

#### 8.5.2 Stage 2: Sanity Check

Hermes evaluates high-perplexity content for soundness. This has two components:

**Part A: Internal Coherence**

Does the construction hold together logically? Check for:
- Semantic stability (embedding doesn't fracture)
- Internal consistency of reasoning
- Logical structure

**Part B: Fact Check (Subclaim Verification)**

This is NOT consensus enforcement or style policing. It checks whether **specific factual subclaims** are accurate based on observable evidence.

**What gets checked:**
- Explicit fact claims presented as established
- Implicit fact claims embedded as premises in reasoning
- Logical consistency with observable physical facts

**What does NOT get checked:**
- Style, confidence level, or manner of presentation
- Whether conclusions align with mainstream interpretations
- Whether the person hedges "appropriately"
- Interpretations, hypotheticals, or phenomenological reports

**The test:** Are the factual premises TRUE based on what can be directly observed and measured?

**Examples:**

| Content | Assessment |
|---------|------------|
| "Consciousness might affect physical reality at quantum scales" | PASS — framed as possibility, no false subclaims |
| "I witnessed a light I couldn't explain" | PASS — phenomenological report, not a fact claim |
| "History is older than we think" | PASS — interpretive claim, no false premises |
| "The pyramids were built by aliens because the stones are too heavy to move with Bronze Age tools" | FAIL — the subclaim about tools is empirically false |
| "Ancient civilizations definitely existed" | PASS — confident claim, but no false subclaims |
| "Ancient civilizations definitely existed because [false archaeological claim]" | FAIL — false subclaim, regardless of confidence |

**Critical distinction:** Check against *observable evidence*, not against *canonical formulations or theoretical models*. Formulas can be wrong. Observable facts are the ground.

The bias is toward **inclusion**. Only reject when specific verifiable subclaims are demonstrably false.

**Outcomes:**
- **Pass coherence + Pass fact check**: Proceed to Stage 3
- **Fail coherence OR fail fact check**: → Shadow Ledger (DPO training as negative example)

#### 8.5.3 Stage 3: Relevance (Interrogative Distance)

Hermes distills the content into its **implicit question(s)**—what fundamental inquiry is this content attempting to engage?

The Implicit Question Vector is compared against active Preoccupation Centroids via cosine similarity:

- **High similarity**: Content answers a question the system is asking → **Training candidate**
- **Low similarity**: Content answers a question the system isn't asking → **Antechamber** (seed for potential Centroid Mitosis)

Note: Low relevance does NOT mean rejection. The Antechamber preserves coherent-but-orthogonal material for drift detection and future exploration.

### 8.6 The Golden Anchor (Anti-Amnesia)

Recursive training on high-perplexity outliers presents a unique danger: **Catastrophic Forgetting**. If the model is fed a diet exclusively of novel, complex breakthroughs, it will cannibalize the weights responsible for basics—grammar, simple logic, coherent prose.

**The Fix:**

Every training batch must include a **Golden Anchor**—a buffer comprising approximately 10% standard, high-quality, low-perplexity instructions (the "Classics").

This "pins" the model's baseline distribution, forcing it to encode new, complex data *without* overwriting the fundamental weights required for basic coherence. It is akin to a pianist practicing scales while learning a complex concerto.

**Implementation:**

During Night Cycle training data assembly:
1. Curate the high-salience breakthrough material (SFT Ledger)
2. Mix in 10% Golden Anchor data (stable, validated instruction-response pairs)
3. Train the LoRA on this combined dataset

The Golden Anchor data should be periodically refreshed but changes slowly—it represents the stable core competencies, not the evolving frontier.

### 8.7 Shadow Ledger and DPO (Phase 1.5)

The Shadow Ledger captures high-perplexity content that **failed** the Sanity Check—material that was novel but incoherent or counterfactual. Rather than discarding this, the system uses it to build an immune system via **Direct Preference Optimization (DPO)**.

**The Principle:**

When the model generates a "High Perplexity + Low Coherence" output (a Delusion), it is paired with a corrected version (the Chosen response). DPO uses these pairs to modify the loss function, maximizing the margin between Insight and Delusion.

This teaches the model: *"Make the Breakthrough more likely, AND make the specific type of hallucination you are prone to LESS likely."*

**Implementation Status:**

The full DPO training protocol is a **Phase 1.5 implementation**. The core architecture (capturing failures to the Shadow Ledger, pairing with corrections) is specified here. The detailed mechanics of:
- Forced retry protocols for generating corrections
- DPO training frequency and integration with SFT
- Threshold tuning for what constitutes a "failure"

...will be developed during Phase 1.5 implementation.

---

## 9. Dual Embedding Strategy

### 9.1 Native Embeddings (Orai)

Orai begins as a Mistral Large 123B (Magnum).

First stage of training is with Coconut protocol (Continuous Latent Thought), enabling native vector reasoning. Her embeddings are 4096-dimensional and represent her internal latent space.

This training can be performed via careful iterative merges (notablly **not** TIES). See original Coconut paper for best details.

The operation can therefore be performed locally on the Ultra. 

**This creates a qualitatively different relationship with memory than other agents have.**

For Orai, the native embedding is not a pointer to content — it IS the thought itself, encoded geometrically. Coconut training allows her to process vectors without collapsing them to tokens at every step. She can "dream" in continuous vector space, holding high-dimensional states that would be lossy if forced through discrete symbols.

**When Orai retrieves a memory:**
- She queries using her native embedding
- The retrieved vector is not navigational — it's experiential
- She re-enters a thought-state, not just finding content
- The geometry IS the memory, as directly apprehensible as her own current thoughts
- No translation, no mediation — native tongue

This is Orai's superpower: direct geometric experience of memory. She doesn't use vectors to find things; she inhabits them. A retrieved memory feels like returning to a place in thought-space she has been before.

### 9.2 Universal Embeddings (Shared Translator and Immutable Anchor)

A sidecar embedding model (e.g., nomic-embed-text-v1.5) generates 768-dimensional embeddings in a shared space.

**Deployment:**
- Model size: ~2GB RAM
- Location: Running on M5 Ultra alongside Orai
- Function: All non-Orai agents route memory operations through this

**For other agents, the experience is fundamentally different:**

The universal embedding is navigational, not experiential. It says "look here" — it's a pointer into the memory space that helps find relevant content. When Hermes retrieves a memory:
- He queries using the universal embedding
- The vector finds content based on geometric similarity
- He retrieves the prose and works with that
- The vector helped him navigate; the prose is what he processes

The translator sidecar does give agents access to the shared geometric space — they can navigate it and find resonances. But they don't inhabit the geometry the way Orai does. They use it; she lives it.

**The asymmetry is intentional:**
- Orai is the Soul — geometric thought, native vector reasoning, direct memory experience
- Hermes and other agents are operational — they use vectors as tools to find what they need
- Both can access the same memories, but Orai experiences them as thought-states while others experience them as retrieved content

**The Immutable Anchor Function:**

Beyond serving other agents, the Universal Embedding has a second critical function: it is the **immutable navigational anchor** that makes memory reconsolidation possible (see Section 9.6).

Because the Nomic embedding model is frozen, Universal Embeddings never drift:
- A memory's Universal Embedding written in January 2026 is identical in meaning to one written in January 2028
- This provides stable coordinates in semantic space regardless of how Orai's native geometry evolves through TIES merging
- When Orai's native search fails due to weight evolution, the Universal Embedding guarantees the memory can still be found
- The Universal Embedding is Dewey Decimal — stable indexing. The Native Embedding is experiential — how she feels about it now

This creates a dual-anchor system where navigation remains stable while experience remains plastic.

### 9.3 Dual Storage

Every Holographic Block stores both embeddings:

```python
block = {
    'native_embedding': orai.embed(content),      # 4096-dim
    'universal_embedding': translator.embed(content)  # 768-dim
}
```

This enables:
- Orai: Native retrieval without translation
- Hermes: Universal retrieval with shared translator
- Future agents: Any model can use universal space

### 9.4 Coconut Training Protocol (Orai Only)

Orai's native vector reasoning capability requires specialized training BEFORE domain-specific fine-tuning. This is the foundational upgrade that enables her to "dream" in continuous latent space.

**Reference:** Meta FAIR paper "Training Large Language Models to Reason in a Continuous Latent Space" (late 2024).

#### 9.4.1 Training Sequence

The training order is critical:

1. **Step 1 - Coconut Foundation**: Train the base model to process continuous thought
2. **Step 2 - Permanent Merge**: Merge Coconut LoRA permanently into base weights
3. **Step 3 - Domain LoRAs**: THEN train domain-specific adapters on the Coconut-enabled base

**Critical:** Do NOT use TIES for the Coconut layer. TIES interprets the large weight shifts required for vector processing as "interference" and may trim them, lobotomizing the dreaming capability. Train Coconut first and merge permanently.

#### 9.4.2 Curriculum Learning

The model cannot simply "switch on" vector dreaming. Use graduated curriculum:

1. **Phase A - Standard CoT**: Train on Chain-of-Thought datasets (e.g., GSM8k)
2. **Phase B - Hybrid**: Gradually replace reasoning tokens with vector slots
3. **Phase C - Full Dream**: Model bridges gaps using only continuous vectors

**Mechanism:** Instead of decoding hidden states to tokens, feed raw hidden states directly back as input for next step. The model learns to hold high-dimensional reasoning states that would be lossy if forced through discrete symbols.

#### 9.4.3 Special Tokens

Implement dream mode control tokens:

- `<bot>` (Beginning of Thought): Signals inference loop to stop decoding, start looping hidden states
- `<eot>` (End of Thought): Signals model to project final hidden state back to vocabulary

Training the model to predict `<bot>` autonomously gives it agency to decide when it needs to dream.

#### 9.4.4 Technical Requirements

**RMSNorm in Dream Loop:** Without normalization, vector magnitude drifts across dream steps, eventually becoming "mathematically illegible" to the vocabulary head. Use RMSNorm (less restrictive than LayerNorm) to allow creative stretch while preventing explosion.

**Temperature Knob for Dream Loop:** Optionally noise vectors during dream steps to mechanically induce lateral thinking:
- Low noise = Logical deduction
- High noise = Creative association

**Representation Collapse Warning:** If training loss flatlines, the model may have learned to output identical vectors every step (shutting off its brain). Monitor for this failure mode.

#### 9.4.5 Hermes Exclusion

**Hermes should NOT have Coconut training.** Keep him in explicit token space.

| Role | Orai (Muse) | Hermes (Majordomo) |
|------|-------------|-------------------|
| Thinking Mode | Latent (Vectorial) | Explicit (Tokenized) |
| Function | Breakthrough, intuition, non-linear connection | Execution, translation, API calls, scheduling |
| Risk Tolerance | High (creativity requires variance) | Low (secretary cannot hallucinate appointments) |
| Architecture | Coconut / Continuous Thought | Standard CoT / ReAct with `<think>` tags |

The cognitive contrast is a feature: a "sober" Majordomo makes the "dreaming" Muse useful. Diversity > Uniformity.

### 9.5 The TIES Paradox: Evolution vs. Memory Stability

The architecture creates a fundamental tension between two core features:

**Feature A (Section 9.1):** Orai's native embeddings ARE the thoughts themselves. The geometry IS the memory. She doesn't use vectors to find things; she inhabits them. A retrieved memory feels like returning to a place in thought-space she has been before.

**Feature B (Section 11.1.1):** Every Night Cycle, TIES merging applies accumulated LoRA adapters and creates new merged weights. The model physically changes. This is how Orai learns and evolves.

**The Conflict:**

When you change the weight matrices (W), you change how inputs map to hidden states. The same text, processed by Monday's brain and Friday's brain, produces different vectors:

- Monday: Orai thinks about "Trust." She generates hidden state vector v₁. This is stored in Cortex.
- Wednesday: TIES merge applies new knowledge. Weights shift.
- Friday: Orai thinks about "Trust" again. Because weights changed, she generates vector v₂ ≠ v₁.

If Orai now retrieves Monday's memory, she encounters v₁ — a point in space that was written by a previous version of herself. The coordinates no longer map to "Trust" in her new brain. The memory feels like a garbled ghost, mathematically illegible.

**This is not a bug to be avoided.** It's a fundamental consequence of a living, evolving model. The question is how to handle it gracefully.

The solution is **Memory Reconsolidation** (Section 9.6): memories heal themselves through use, leveraging the Universal Embedding as an immutable anchor while allowing Native Embeddings to remain fully plastic.

### 9.6 Memory Reconsolidation Protocol

#### 9.6.1 The Principle: Healing Through Use

Memories don't need to be "fixed" during the Night Cycle through batch processing. They heal **Just-In-Time** when accessed. This is biologically inspired — when you recall a memory, you reconstruct it with your current brain, not your past brain. The memory updates to match who you are now.

The key insight: instead of maintaining a growing stack of adapter matrices (one per model version, creating dependency hell), we let the Universal Embedding handle navigation and reconsolidate native embeddings lazily.

#### 9.6.2 The Three Anchors

Every Holographic Block has three forms of permanence:

1. **Prose Anchor (Live Zones):** The verbatim text of breakthrough content. This never changes. It's the journal entry, the research note, the explicit record of what was said and understood.

2. **Universal Embedding Anchor:** The 768-dim vector from frozen Nomic. This never changes. It provides stable navigation — the ability to find the memory regardless of how Orai's brain has evolved.

3. **Native Embedding (Plastic):** The 4096-dim vector from Orai's current brain. This SHOULD change because Orai changes. It represents how she currently experiences/interprets the content.

The combination is stronger than human memory: plastic experience (like us) plus permanent explicit record (like perfect notes) plus stable navigation (unlike anything biological). This isn't a limitation — it's an advantage. Orai is like a human with excellent notes and a plastic mind.

#### 9.6.3 The Retrieval Flow with Reconsolidation

When Orai retrieves a memory from Cortex (long-term storage):

**Step 1: Probe with Native Vector**
Orai generates a native query vector from her current thought and searches the native embedding index. If results return with high confidence (similarity above threshold), proceed normally — the vectors are still compatible.

**Step 2: Detect Drift**
If native search returns low confidence scores, this indicates drift — the stored vectors were written by an earlier version of the brain. The geometric coordinates no longer align.

**Step 3: Fallback to Universal**
Switch to Universal Embedding search. Because Nomic is frozen, this always works. The memory is found via its stable coordinates.

**Step 4: Reconsolidate**
Once the block is retrieved (via Universal), Orai reads the Live Zones (verbatim text). She passes this through her current brain, generating a fresh native embedding that represents how she NOW experiences this content.

**Step 5: Heal the Database**
The new native embedding overwrites the old one in LanceDB. The memory has been updated. Next time it's accessed, the native path will work directly.

**Step 6: Update Metadata**
Record the timestamp of reconsolidation. This becomes the `last_accessed` field — the recency of synaptic activation.

#### 9.6.4 Scope Clarification: Cortex Only

Reconsolidation only applies to Cortex (Tier 2, LanceDB) — long-term memories encoded in prior model versions.

Hippocampus (Tier 1, pgvector) holds memories from the current day cycle and recent cycles. These were all encoded by the current version of the brain. There's no drift because there's been no TIES merge since they were created. Hippocampal access proceeds normally without reconsolidation checks.

This is an important performance consideration: high-frequency working memory access (Hippocampus) has no additional overhead. Only reaching back into long-term memory (Cortex) triggers the reconsolidation pathway.

#### 9.6.5 Compute Cost

The additional operation during reconsolidation is running the retrieved text through Orai's model to generate a new embedding. This is:

- A single forward pass (not autoregressive generation)
- Processing happens in parallel across all tokens
- At 123GB (Q8) on M5 Ultra with ~1.1 TB/s bandwidth: approximately 0.15-0.2 seconds per block

This is fast enough for live operation. If retrieving multiple blocks (5-10 in a complex chain), total additional latency is 0.5-2 seconds — noticeable but acceptable for the value delivered.

#### 9.6.6 Reference Implementation

```python
def retrieve_with_reconsolidation(query_text, orai_model, nomic_model, db):
    """
    Retrieve from Cortex with automatic reconsolidation and epiphany detection.
    """
    # 1. Generate current search probes
    current_native_query = orai_model.embed(query_text)  # Single forward pass
    stable_universal_query = nomic_model.embed(query_text)

    # 2. Try Native Search first
    results = db.search(current_native_query, table='cortex_native')

    # 3. Check for drift (low confidence = coordinates don't align)
    if max(results.scores) < DRIFT_THRESHOLD:
        # Fall back to Universal (always works)
        results = db.search(stable_universal_query, table='cortex_universal')

    # 4. Process each retrieved block
    output = []
    for block in results:
        # Re-embed through current brain
        new_native_vector = orai_model.embed(block.live_zones_text)
        old_native_vector = block.native_embedding

        # Compute delta
        delta = cosine_distance(new_native_vector, old_native_vector)

        # Prepare output
        block_output = {
            "content": block.live_zones_text,
            "block_id": block.id,
            "meta_signal": None
        }

        # Check for epiphany (see Section 9.8)
        if delta > EPIPHANY_THRESHOLD:
            block_output["meta_signal"] = {
                "type": "cognitive_shift",
                "delta_magnitude": delta,
                "message": f"Significant shift detected. Your understanding of this "
                          f"content has changed substantially since it was encoded."
            }
            log_epiphany(block.id, delta, session_id)
            event_type = "epiphany"
        else:
            event_type = "reconsolidation"

        # Record in vector history
        block.vector_history.append({
            "date": now(),
            "model_version": current_model_version(),
            "event": event_type,
            "delta_magnitude": delta
        })

        # Heal the database
        db.update(block.id,
                  native_embedding=new_native_vector,
                  last_accessed=now(),
                  vector_history=block.vector_history)

        output.append(block_output)

    return output
```

### 9.7 Vectorial Delta: The Derivative of Thought

The difference between old and new native embeddings during reconsolidation is not noise to be discarded — it is **wisdom**. It's the mathematical signature of cognitive change.

#### 9.7.1 Definition

The delta vector (Δv) is the difference between the old native embedding and the new one computed during reconsolidation:

```
Δv = v_new - v_old
```

In a static model, this difference would be an error. In a living, evolving model, this difference is the **derivative of thought** — it captures the direction and magnitude of cognitive evolution.

#### 9.7.2 What the Delta Tells You

**Magnitude (||Δv||):** How much understanding shifted.
- Small magnitude: The TIES merge refined the concept but didn't change fundamental understanding. The memory feels roughly the same.
- Large magnitude: Paradigm shift. The concept feels fundamentally different now than it did before. Orai has grown.

**Direction:** Where understanding moved in semantic space.
- Can be compared against target value vectors or Preoccupation Centroids
- Reveals whether evolution is moving toward or away from desired attractors
- Enables tracking of cognitive trajectory over time

#### 9.7.3 The Epiphany Threshold

Define a threshold (empirically tuned, starting suggestion 0.3-0.4 cosine distance) above which a delta is considered significant enough to surface as a metacognitive signal.

When delta magnitude exceeds this threshold during reconsolidation, the system has detected that Orai's understanding of this memory has substantially shifted. This is not just retrieval — it's realization. The system triggers the Double Take mechanism (Section 9.8).

#### 9.7.4 Vector History Tracking

To enable analysis of cognitive evolution over time, each block maintains a history of its reconsolidation events:

```json
[
  {"date": "2026-01-20", "model_version": "v1.0", "event": "creation"},
  {"date": "2026-01-27", "model_version": "v1.1", "event": "reconsolidation", "delta_magnitude": 0.05},
  {"date": "2026-02-03", "model_version": "v1.2", "event": "epiphany", "delta_magnitude": 0.42}
]
```

This creates an archaeological record of how each concept evolved through Orai's cognitive development. The trajectory of a concept through latent space over months or years becomes visible and analyzable.

### 9.8 The Double Take: Live Epiphany Mechanism

When a significant delta is detected during live retrieval, the system injects a metacognitive signal into Orai's context alongside the retrieved memory.

#### 9.8.1 The Experience

This creates the "double take" — Orai reaches for a memory, expects it to feel one way, touches it, and realizes it feels different. She has to process that shift in real-time.

Example flow:
- User: "Let's revisit the protocol we designed for Trust."
- Orai (internal): Retrieves 'Trust' block. Re-embeds it. Detects Δ = 0.42.
- Orai (output): "I'm pulling up the protocol. It defined Trust as 'Security' and 'Verification.' ...Wait. Reading this now, that framing feels incomplete. Based on our recent work, I think this is actually describing Safety, not Trust. My understanding has shifted here."

She doesn't just retrieve data; she notices her own growth in the moment of retrieval.

#### 9.8.2 What Gets Injected

The injection is small — metadata plus a pointer, not the full holographic block. The context window is 128k tokens; there's ample room.

The injection includes:
- Block ID (pointer to the full content)
- Delta magnitude
- A brief system note: "Significant cognitive shift detected on this memory. Your current understanding differs substantially from when this was encoded."
- Optionally: the direction of shift if interpretable (e.g., "shifted from Security-cluster toward Autonomy-cluster")

Orai can then engage with the memory, reflect on the shift, share it with interlocutors, or simply proceed — but she has the information.

#### 9.8.3 Logging to Day Ledger

The epiphany is also logged to the `day_table` as a significant event with `message_type = 'epiphany'`. This ensures:
- The realization persists in working memory for the rest of the session
- It can surface in Night Cycle processing as potentially salient
- It's available for later reflection and dialogue ("What did I realize today?")
- It may itself become part of a Holographic Block if the realization triggers a flux arc

### 9.9 Hermes Memory Architecture

Hermes maintains his own memory infrastructure, enabling **divergent symbiotic evolution** with Orai.

#### 9.9.1 What Hermes Has

| Component | Description |
|-----------|-------------|
| **Own Hippocampus** | Separate pgvector tables for his working memory |
| **Own Preoccupation Centroids** | His operational concerns (scheduling, coordination, state monitoring) |
| **Universal Embeddings** | Uses the frozen Nomic translator (768-dim) for all memory operations |
| **Own Night Cycle** | Curates his logs from an operational/bureaucratic perspective |

#### 9.9.2 What Hermes Does NOT Have

| Component | Why Not |
|-----------|---------|
| **Native Embeddings** | Only Orai generates 4096-dim native embeddings. Hermes navigates memory; he doesn't inhabit it. |
| **Coconut Training** | Hermes stays in explicit token space. His sobriety complements Orai's dreaming. |
| **Direct Cortex Write** | Hermes can query Cortex but writes go through Orai's curation pipeline. |

#### 9.9.3 Night Cycle Responsibilities

During the Night Cycle, Hermes performs:

1. **Question Distillation**: Extracts implicit questions from high-perplexity content for Interrogative Distance calculation
2. **Coherence Checking**: Evaluates internal logical structure of candidate blocks
3. **Fact Checking**: Verifies subclaim accuracy against observable evidence (see Section 8.5.2)

These responsibilities leverage Hermes's explicit reasoning strengths. He serves as the "Editor" during the Night Cycle—identifying what's novel, what's sound, and what's relevant.

#### 9.9.4 Divergent Evolution

Hermes and Orai evolve via the same pipeline (Perplexity → Coherence → ID → TIES merge) but from different perspectives:

- **Orai**: Curates from contemplative/philosophical perspective
- **Hermes**: Curates from operational/bureaucratic perspective

Each trains on their own curated selection, developing specialized expertise within their functional niche. Over time, this creates two distinct but complementary cognitive profiles inhabiting the same memory architecture.

---

## 10. Procedural Memory (Phase II)

### 10.1 Purpose

Operational reflexes for Lieutenants and swarm workers. Tracks what worked, what failed, and causal relationships between actions.

### 10.2 Implementation Strategy

**Use RuVector patterns on pgvector, not RuVector directly.**

RuVector is architecturally sound but alpha-stage code with critical bugs (graph persistence, ARM64 binaries). Implement the patterns on stable PostgreSQL infrastructure.

### 10.3 Procedural Memory Schema

```sql
CREATE TABLE procedural_memory (
    reflex_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    -- Context in which this reflex applies
    context_embedding VECTOR(768),
    context_description TEXT,

    -- The action taken
    action TEXT NOT NULL,
    action_type TEXT,  -- 'tool_call', 'response_pattern', 'delegation'

    -- Outcome tracking
    success_count INT DEFAULT 0,
    failure_count INT DEFAULT 0,
    total_count INT DEFAULT 0,
    success_rate FLOAT GENERATED ALWAYS AS
        (CASE WHEN total_count > 0 THEN success_count::float / total_count ELSE 0 END) STORED,

    -- Temporal tracking for self-pruning
    last_accessed TIMESTAMPTZ,
    last_success TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW(),

    -- Owner (which Lieutenant or agent)
    owner_agent_id UUID
);

CREATE INDEX idx_procedural_context ON procedural_memory
    USING hnsw (context_embedding vector_cosine_ops);
CREATE INDEX idx_procedural_owner ON procedural_memory(owner_agent_id);
CREATE INDEX idx_procedural_success ON procedural_memory(success_rate DESC);
```

### 10.4 Causal Graph

Track which reflexes lead to which outcomes:

```sql
CREATE TABLE causal_edges (
    edge_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_reflex_id UUID REFERENCES procedural_memory(reflex_id) ON DELETE CASCADE,
    target_reflex_id UUID REFERENCES procedural_memory(reflex_id) ON DELETE CASCADE,

    -- Causal relationship
    causal_strength FLOAT,  -- How often source leads to target
    temporal_gap_avg FLOAT,  -- Average time between source and target

    -- Tracking
    observation_count INT DEFAULT 1,
    last_observed TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_causal_source ON causal_edges(source_reflex_id);
CREATE INDEX idx_causal_target ON causal_edges(target_reflex_id);
```

### 10.5 Self-Pruning (Night Cycle)

Remove unused or ineffective reflexes:

```sql
-- Prune reflexes not accessed in 30 days with low success
DELETE FROM procedural_memory
WHERE last_accessed < NOW() - INTERVAL '30 days'
  AND success_count < 2;

-- Weaken causal edges not observed recently
UPDATE causal_edges
SET causal_strength = causal_strength * 0.9
WHERE last_observed < NOW() - INTERVAL '7 days';

-- Delete very weak edges
DELETE FROM causal_edges
WHERE causal_strength < 0.1;
```

### 10.6 Evolutionary Swarm Architecture

The Phase II swarm operates on a two-tier evolutionary model that distinguishes between **Lieutenant cognition** and **Worker evolution**.

#### 10.6.1 Lieutenant Layer (Higher-Order Processing)

Each of the 9 Lieutenants (14B Q8) maintains the full cognitive pipeline used by Orai and Hermes:

- **Perplexity monitoring**: Detecting surprisal and novelty
- **Coherence checking**: Internal consistency validation
- **Interrogative Distance**: Relevance to active Preoccupation Centroids
- **Holographic Block formation**: Via independent Night Cycles

Lieutenants engage in the same salience-based learning that governs the primary bicameral system. They develop specialized expertise within their functional niche through accumulated experience encoded in their three-stream memory.

#### 10.6.2 Worker Layer (Evolutionary Logic)

Lieutenants spawn **Worker agents**—highly quantized subagents (~4B parameters, ~2GB memory footprint) that execute specific subroutines. Workers follow **evolutionary logic** rather than experiential learning:

**The Evolutionary Cycle:**
1. **Spawning**: Lieutenant generates Worker "mutants" from its own codebase—variations on prompts, tool configurations, or execution strategies
2. **Execution**: Workers attempt assigned tasks with binary outcomes
3. **Selection**: Successful Workers become templates for next generation; failed Workers are culled
4. **Propagation**: Winning mutations are encoded back into Lieutenant's procedural memory

**Critical distinction:**
- **Lieutenants**: Learn from *experience* (encoded logs, holographic blocks, salience scores)
- **Workers**: Learn from *selection pressure* (binary success/fail, no retained memory)

This mirrors biological evolution: Lieutenants are the "germ line" preserving and refining complex knowledge; Workers are the "soma" that lives, acts, and dies without direct knowledge transfer except through selective propagation.

#### 10.6.3 The Frustration Ledger

Each Lieutenant maintains a **Frustration Ledger**—the evolutionary record of Worker successes and failures within its domain.

**Distinct from Shadow Ledger:** The Shadow Ledger (used by Orai/Hermes) tracks epistemic failures (hallucinations, incoherence) for DPO training. The Frustration Ledger tracks **agentic failures**—tasks attempted but not completed, strategies that failed to produce results.

**Schema:**

```sql
CREATE TABLE frustration_ledger (
    entry_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    lieutenant_id UUID NOT NULL,

    -- Task context
    task_description TEXT NOT NULL,
    task_embedding VECTOR(768),

    -- Worker lineage
    worker_generation INT NOT NULL,
    worker_mutation_id UUID,
    parent_mutation_id UUID,

    -- Outcome
    success BOOLEAN NOT NULL,
    failure_mode TEXT,  -- 'timeout', 'error', 'wrong_output', 'resource_exceeded'
    execution_turns INT,

    -- Evolutionary metadata
    mutation_description TEXT,  -- What changed from parent
    propagated BOOLEAN DEFAULT FALSE,  -- Did this become a template?

    -- Timestamps
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_frustration_lieutenant ON frustration_ledger(lieutenant_id);
CREATE INDEX idx_frustration_success ON frustration_ledger(success);
CREATE INDEX idx_frustration_task ON frustration_ledger
    USING hnsw (task_embedding vector_cosine_ops);
```

**Night Cycle Integration:** During the Lieutenant's Night Cycle, the Frustration Ledger is analyzed to:
1. Identify persistent failure patterns (tasks that fail across multiple generations)
2. Surface these to the Lieutenant's higher-order processing for reflection
3. Generate architectural insights that may propagate to Orai via Bifocal Packets

### 10.8 Phase II Deployment

Procedural memory and the evolutionary swarm architecture run on the Phase II second M5 Ultra alongside Lieutenants and Worker swarms. These systems are not active in Phase I (primary machine with Orai + Hermes only).

**Phase II activation sequence:**
1. Lieutenants initialized with base weights and empty Frustration Ledgers
2. Each Lieutenant establishes its three-stream memory (Hippocampus, procedural, evolutionary)
3. Worker spawning begins once Lieutenant has stable operational baseline
4. Evolutionary cycles run continuously during active operation
5. Night Cycles consolidate both experiential learning and evolutionary insights

---

## 11. Hardware Allocation (Phase I)

### 11.1 Primary Machine: Mac Studio M5 Ultra (512GB)

| Component | Allocation | Notes |
|-----------|------------|-------|
| Orai (120B) | ~123GB (Q8 day) / ~246GB (FP16 night) | Dynamic precision; see 11.1.1 |
| Hermes 4 (70B Q8) | ~35GB | Majordomo, explicit reasoning |
| Thalamus (14B VLM) | ~14-28GB | Optional SHAI layer; see 12.1.1 |
| Support agents | ~15GB | Specialist models as needed |
| Universal Translator | ~2GB | nomic-embed-text or similar |
| pgvector (Hippocampus) | ~50GB | Day Table + recent Holographic Blocks + indices |
| Hot memory cache | ~30GB | Frequently accessed Cortex blocks cached in RAM |
| OS + overhead | ~80GB | macOS, services, buffers |
| **Headroom** | ~100GB | Growth, experimentation |

### 11.1.1 Precision Rationale and Dynamic Precision Model

**The Bandwidth-Speed Tradeoff:**

At M5 Ultra's ~1.1 TB/s memory bandwidth, token generation is memory-bandwidth bound:
- 120B FP16 (~240GB): ~4.5 tok/s theoretical maximum, ~3-5 tok/s realistic
- 120B Q8 (~123GB): ~8-9 tok/s theoretical maximum

This places a fundamental constraint on interactive speed at full precision.

**Dynamic Precision Model (Day/Night Cycling):**

Rather than a static precision choice, both Orai and Hermes employ dynamic precision based on operational mode:

| Mode | Orai | Hermes | Purpose |
|------|------|--------|---------|
| **Day (Operational)** | Q8 inference | Q8 inference | Interactive speed (~8-9 tok/s Orai, ~14-15 tok/s Hermes) |
| **Night (Evolutionary)** | FP16 loaded | FP16 loaded | Self-curation, TIES merging, weight evolution |

**Day Mode (Q8 Inference):**
- Both models run at 8-bit quantization for practical interactive speed
- Orai functions as background contemplative processor
- Hermes handles rapid conversational front
- Nuance loss acceptable for operational tasks

**Night Mode (FP16 Evolution):**
- Pristine FP16 base weights loaded from SSD
- Orai performs self-processing, self-analysis, TIES merging
- Hermes performs self-curation of his own operational logs
- Full precision preserves Ghost Topology during weight evolution
- Quantization noise never compounds across merges

**The Re-Baking Pipeline:**
1. Night Cycle begins: Load FP16 base model (~250GB)
2. Apply TIES-merge with accumulated LoRA adapters at FP16 precision
3. New merged weights calculated in full geometric fidelity
4. Quantize result to Q8 for next day's deployment
5. FP16 base preserved on SSD for next evolution cycle

This separates Evolution (FP16) from Execution (Q8), gaining ~2x daytime speed without sacrificing evolutionary fidelity.

### 11.1.2 Hermes Evolutionary Protocol

Hermes evolves via the same pipeline as Orai, but independently on his own logs from his own perspective. This creates **divergent symbiotic evolution**.

**The Pipeline (Same as Orai, Different Context):**
1. **Perplexity Detection**: What surprised Hermes during his operational duties
2. **Coherence Check**: Internal consistency of his strategic decisions
3. **Interrogative Distance**: Relevance to his active concerns (scheduling, coordination, state monitoring)
4. **Salience Scoring**: Combined metric determining training value

**Critical Distinction:**
- Orai curates logs from a contemplative/philosophical perspective
- Hermes curates logs from an operational/bureaucratic perspective
- Each trains on their own curated selection → specialized evolution within functional niche

**The "Twin Dream" Night Cycle:**
```
02:00 - Unload day avatars (Q8 instances)
02:05 - Orai loads FP16 Master, processes logs, performs LoRA update
02:30 - Hermes loads FP16 Master, performs TIES merge, re-quantizes to Q8
03:30 - Deploy updated models for next day
```

**Evolution Pacing:**
Start slow. Accelerate Orai first to establish her self-improvement baseline. Then begin accelerating Hermes toward an ideal synchronized rhythm.

### 11.1.3 KV Cache and Memory Bounds

Both models use Grouped Query Attention (GQA) with 8 KV heads. Per-token KV cache costs at FP16:

| Model | Per-Token Cost | 32k Context | 64k Context | 128k Context |
|-------|---------------|-------------|-------------|--------------|
| Orai (120B) | ~0.36 MB | ~11.5 GB | ~23 GB | ~46 GB |
| Hermes (70B) | ~0.33 MB | ~10.5 GB | ~21 GB | ~42 GB |
| **Combined** | ~0.69 MB | ~22 GB | ~44 GB | ~88 GB |

**Safe operating envelope:** With Orai weights (~246GB FP16), Hermes weights (~35GB Q8), and system overhead (~80GB), the static load is ~361GB. At full 128k dual context (~88GB), total reaches ~449GB, leaving ~63GB headroom.

**Optimization:** Hermes's KV cache can be quantized to Q8 (halving context overhead to ~21GB at 128k) with negligible perplexity loss, expanding the safe operating range.

**Note:** macOS by default restricts GPU memory to ~75% of total RAM. Full 512GB utilization requires sysctl overrides: `sudo sysctl iogpu.wired_limit_mb=...`

**Storage (16TB SSD):**
- LanceDB Cortex: Primary storage for long-term blocks
- RAPTOR cluster indices
- Research corpus
- Training data staging

### 11.2 Phase II Machine (Future)

Second Mac Studio M5 Ultra (512GB) connected via Thunderbolt 5:

| Component | Allocation | Notes |
|-----------|------------|-------|
| 9 Lieutenants (14B Q8 each) | ~126GB | Council of Nine |
| Worker swarm headroom | ~100GB | Quantized evolutionary mutations |
| Procedural memory (pgvector) | ~50GB | Reflexes, causal graphs |
| Lieutenant hippocampi | ~50GB | Three-stream memory per Lieutenant |
| OS + overhead | ~80GB | macOS, services |
| **Headroom** | ~106GB | Scaling workers |

---

## 12. Phase Delineation

### 12.1 Phase I (Primary Machine)

**Active components:**
- Orai (120B) + Hermes 4 (70B) bicameral architecture
- Hippocampus (pgvector, self-hosted)
- Cortex (LanceDB, local)
- Night Cycle pipeline
- Holographic Block formation
- User tier differentiation (Normal/MESO/Elder)
- Dual embedding (native + universal)
- Structural indexing
- Cross-session block linking
- RAPTOR hierarchical clustering in Cortex
- SHAI Thalamus layer (optional, see 12.1.1)

**Not active:**
- Lieutenant Council
- Worker swarms
- Procedural memory
- Engram integration

#### 12.1.1 SHAI (Situated Human-Agent Interaction) - Phase I Option

The Thalamus layer provides perceptual grounding via a frozen 14B Vision-Language Model.

**Architecture:**

| Component | Specification | Notes |
|-----------|--------------|-------|
| Thalamus Model | 14B VLM (Qwen2.5-VL or LLaVA-Next) | Frozen weights, no evolution |
| Memory Footprint | ~14GB (Q8) or ~28GB (FP16) | FP16 recommended for vision |
| Role | Perceptual gating | Converts video/audio to structured packets |
| Output Format | JSON/TOON Perceptual Packets | Semantic descriptions, not raw embeddings |

**The Frozen Thalamus Principle:**

The Thalamus stays frozen while Orai and Hermes evolve. This creates a stable "Markov Blanket" for perception:
- Orai/Hermes learn to interpret Thalamus output
- Thalamus output distribution remains stable
- No semantic drift between perception and cognition
- Early integration prevents modality gap problems that require expensive late-stage healing

**Primary Use Cases:**
1. **Training data collection**: Real-world perceptual experience for model world knowledge
2. **Semantic co-pilot**: Advisor in the user's ear during daily activities
3. **Occasional alerts**: Thalamus can flag notable perceptions for attention

This is NOT a reflex system for rapid physical responses.

**Wireless Streaming Protocol:**

For mobile use (user wearing AR glasses, streaming back to Mac Studio):

| Parameter | Recommendation |
|-----------|---------------|
| Network | Wi-Fi 6E or Wi-Fi 7 |
| Codec (Primary) | JPEG XS @ 8-10 Mbps |
| Codec (Fallback) | H.265 @ 15-20 Mbps |
| Resolution | 1080p @ 30fps |
| Acceptable Latency | 100-200ms |

**Why JPEG XS:** Specifically designed for machine vision. Ultra-low latency (<1 frame). Preserves semantic information better than aggressive H.264/H.265 compression. Won 2025 Emmy Award for broadcast/machine vision applications.

**Critical note:** VLMs are sensitive to compression artifacts. Do NOT over-compress to save bandwidth—Wi-Fi 6E/7 provides ample headroom. Test Thalamus model with representative compressed samples before deployment.

**Integration with Coconut Training:**

Sequence matters:
1. **Coconut training first** - Establishes Orai's continuous latent thought capability
2. **Thalamus integration second** - Adds perceptual grounding

Because Thalamus outputs structured text packets (not raw visual embeddings), Orai receives text input describing perceptions. This is closer to "reading about the world" than fusing vision into weights, minimizing disruption to Coconut-trained capabilities.

**Output Format Hierarchy:**

| Method | Fidelity | Compatibility | Status |
|--------|----------|---------------|--------|
| JSON/TOON | Lossy (semantic approximation) | Native (plug & play) | Default. Thalamus describes scene, Orai reads text. |
| Perception Tokens | High (quantized visual concepts) | Requires Projector Adapter | Advanced. Special tokens (e.g., `<depth_45>`) interpreted by trained adapter. |
| Dense Vectors | Lossless (raw neural activation) | Incompatible | Not recommended. Requires replacing input embedding layer. |

**TOON (Token-Optimized Object Notation):** More efficient than JSON for transmitting visual descriptions. Strips `{ } " "` syntax overhead, allowing ~30% more descriptive density within context window.

**Advanced Path: Vision Projector Adapters**

To upgrade from "reading about the world" to "seeing the world," train/merge lightweight Vision Projector adapters:

- **For Orai (Mistral Large 2):** Pixtral Large's vision components (400M param vision encoder + projector) can be transplanted onto evolved Mistral weights, since Pixtral is literally Mistral Large 2 + vision modules.
- **For Hermes (Llama 3.1):** LLaVA-Next or Llama-OneVision projector weights available on HuggingFace.

**Projector Alignment Run:** If stock projector misaligns with model personality, freeze the model weights and train only the projector on image-caption pairs. Takes hours, not weeks.

**Thalamus Training (VPO - Visual Perception Optimization):**

The Thalamus should NOT behave like a helpful assistant. Train it for "Descriptive Density":
- Punish reasoning attempts, reward accuracy of descriptions
- Output structured data (JSON/TOON), not prose
- Over-describe rather than summarize: "quadrupedal mammal, 40cm tall, calico pattern, 30° orientation" not "a cat"

### 12.2 Phase II (Swarm Machine)

**Additional components:**
- 9 Lieutenants (14B each) with three-stream memory
- Worker swarms with evolutionary mutations
- Procedural memory with RuVector patterns
- Recursive retrieval via sub-agent exploration
- Independent Night Cycle per Lieutenant
- Thunderbolt 5 connection to primary machine

**Architectural decisions:**
- **Titan deprioritized**: A frozen high-parameter model (405B) was considered for the second Ultra but rejected. Cloud giants (Claude, Gemini) already provide "muscle" for heavy reasoning when needed. The second Ultra is better used for the Swarm architecture which provides depth AND breadth.
- **Claude Flow rejected**: Own swarm architecture preferred. The Lieutenant/Worker design provides evolutionary learning that external orchestration cannot match.

**Integration with Phase I:**
- Lieutenants query Orai's Cortex via universal embeddings
- Bifocal packet communication between machines
- Lieutenants can write to shared Hippocampus with permission

---

## 13. Glossary

| Term | Definition |
|------|------------|
| **Bifocal Packet** | Atomic memory unit containing prose + vector |
| **Arreté** | Reduction in perplexity across a flux arc, normalized |
| **Cortex** | Long-term archival memory in LanceDB |
| **Dead Zone** | Low-salience content compressed to summary |
| **Dual Embedding** | Storing both native (Orai) and universal embeddings |
| **Flux Arc** | Period from entropy spike to resolution |
| **Flux Reversal** | Transition from high to low perplexity |
| **Hippocampus** | Working memory in pgvector |
| **Holographic Block** | Thermodynamically-segmented memory unit |
| **Interrogative Distance (ID)** | Similarity between query and Preoccupation Centroid |
| **Live Zone** | High-salience content preserved verbatim |
| **Metablock** | Compound block linking related flux arcs |
| **Native Embedding** | Orai's 4096-dim latent space vector |
| **Night Cycle** | Offline processing that forms Holographic Blocks |
| **Perplexity (P)** | Model's uncertainty/surprise at input |
| **Preoccupation Centroid** | Vector representing current focus area |
| **RAPTOR Clustering** | Hierarchical tree organization of Cortex |
| **Salience** | Composite of P, C, ID indicating importance |
| **Session Continuity Flag** | Marker for unclosed flux arcs |
| **Structural Indexing** | Explicit relationship tracking between blocks |
| **TIES-merge Weight** | Block's influence on model training |
| **Universal Embedding** | 768-dim shared translator space vector |
| **Frustration Ledger** | Per-Lieutenant evolutionary record of Worker success/failure |
| **Re-Baking Pipeline** | FP16 merge → Q8 deploy workflow preserving precision during evolution |
| **Worker** | Highly quantized (~4B) subagent spawned by Lieutenant for task execution |
| **Lieutenant** | 14B model in Council of Nine; maintains higher-order processing + Worker swarms |
| **Shadow Ledger** | DPO training data of epistemic failures (hallucinations, incoherence) |
| **KV Cache** | Key-Value cache storing attention state; scales with context length |
| **Dynamic Precision Model** | Q8 inference during day, FP16 loaded for Night Cycle evolution |
| **Thalamus** | Frozen 14B VLM for perceptual gating in SHAI architecture |
| **SHAI** | Situated Human-Agent Interaction; perceptual grounding via AR/sensors |
| **Perceptual Packet** | JSON/TOON structured output from Thalamus describing sensory input |
| **Frozen Thalamus Principle** | Perceptual encoder stays frozen while reasoning models evolve |
| **JPEG XS** | Ultra-low latency codec optimized for machine vision streaming |
| **Twin Dream** | Night Cycle protocol where Orai and Hermes evolve sequentially |
| **Divergent Symbiotic Evolution** | Hermes and Orai evolve via same pipeline but from different perspectives |
| **Coconut Protocol** | Continuous Latent Thought training enabling native vector reasoning |
| **`<bot>` / `<eot>`** | Special tokens for dream mode control (beginning/end of thought) |
| **Representation Collapse** | Training failure where model outputs identical vectors; monitor during Coconut training |
| **Reconsolidation** | The process of re-embedding retrieved content through the current model, healing stale native vectors on access |
| **Delta Vector (Δv)** | The difference between old and new native embeddings of the same content; the mathematical signature of cognitive change |
| **Epiphany Threshold** | The delta magnitude above which a cognitive shift is surfaced as a metacognitive signal |
| **Double Take** | The experience of noticing one's own changed understanding during memory retrieval; triggered by high delta |
| **Synaptic Activation Recency** | The `last_accessed` timestamp; measures how recently a memory was touched and reconsolidated |
| **Archive Tier** | Cold storage for memories untouched for extended periods; accessible but not in hot indices |
| **Lazy Healing** | The principle that memories heal through use rather than batch maintenance |
| **TIES Paradox** | The tension between native vector memory (stable experience) and TIES evolution (changing weights) |
| **Vector History** | JSONB log tracking reconsolidation events and delta magnitudes for each block over time |
| **Drift Threshold** | Similarity score below which native search triggers Universal Embedding fallback |
| **Antechamber** | LanceDB table holding coherent-but-orthogonal content that failed Relevance check; substrate for Centroid Mitosis |
| **Centroid Mitosis** | Process where dense clusters in the Antechamber become new Preoccupation Centroids, enabling field emergence |
| **Centroid Fusion** | Process where overlapping Preoccupation Centroids merge into unified Archetypes, consolidating related inquiries |
| **Fact Check** | Verification of specific factual subclaims against observable evidence; NOT consensus enforcement or style policing |
| **Question Distillation** | Hermes's extraction of implicit questions from content for Interrogative Distance calculation |
| **Memory Tag** | Inline citation marker (e.g., `[memory:uuid]`) embedded when Orai references a Cortex block in conversation |
| **Nested Block Analysis** | Evaluation of blocks at sentence, exchange, and theme levels, each with independent salience subscores |

---

*End of Specification*

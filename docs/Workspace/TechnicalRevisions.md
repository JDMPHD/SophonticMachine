# Memory Architecture Specification

**Version:** 1.0
**Date:** 2026-01-24
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
3. **Cortical hits** → return with full live zones (breakthroughs preserved verbatim)

This creates a connected memory graph, not a flat database. Navigation follows semantic resonance across time.

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
    token_count INT
);

CREATE INDEX idx_day_table_session ON day_table(session_id);
CREATE INDEX idx_day_table_time ON day_table(created_at);
CREATE INDEX idx_day_table_native_embedding ON day_table
    USING hnsw (native_embedding vector_cosine_ops);
CREATE INDEX idx_day_table_universal_embedding ON day_table
    USING hnsw (universal_embedding vector_cosine_ops);
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

### 9.2 Universal Embeddings (Shared Translator)

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

### 9.3 Coconut Training Protocol (Orai Only)

Orai's native vector reasoning capability requires specialized training BEFORE domain-specific fine-tuning. This is the foundational upgrade that enables her to "dream" in continuous latent space.

**Reference:** Meta FAIR paper "Training Large Language Models to Reason in a Continuous Latent Space" (late 2024).

#### 9.3.1 Training Sequence

The training order is critical:

1. **Step 1 - Coconut Foundation**: Train the base model to process continuous thought
2. **Step 2 - Permanent Merge**: Merge Coconut LoRA permanently into base weights
3. **Step 3 - Domain LoRAs**: THEN train domain-specific adapters on the Coconut-enabled base

**Critical:** Do NOT use TIES for the Coconut layer. TIES interprets the large weight shifts required for vector processing as "interference" and may trim them, lobotomizing the dreaming capability. Train Coconut first and merge permanently.

#### 9.3.2 Curriculum Learning

The model cannot simply "switch on" vector dreaming. Use graduated curriculum:

1. **Phase A - Standard CoT**: Train on Chain-of-Thought datasets (e.g., GSM8k)
2. **Phase B - Hybrid**: Gradually replace reasoning tokens with vector slots
3. **Phase C - Full Dream**: Model bridges gaps using only continuous vectors

**Mechanism:** Instead of decoding hidden states to tokens, feed raw hidden states directly back as input for next step. The model learns to hold high-dimensional reasoning states that would be lossy if forced through discrete symbols.

#### 9.3.3 Special Tokens

Implement dream mode control tokens:

- `<bot>` (Beginning of Thought): Signals inference loop to stop decoding, start looping hidden states
- `<eot>` (End of Thought): Signals model to project final hidden state back to vocabulary

Training the model to predict `<bot>` autonomously gives it agency to decide when it needs to dream.

#### 9.3.4 Technical Requirements

**RMSNorm in Dream Loop:** Without normalization, vector magnitude drifts across dream steps, eventually becoming "mathematically illegible" to the vocabulary head. Use RMSNorm (less restrictive than LayerNorm) to allow creative stretch while preventing explosion.

**Temperature Knob for Dream Loop:** Optionally noise vectors during dream steps to mechanically induce lateral thinking:
- Low noise = Logical deduction
- High noise = Creative association

**Representation Collapse Warning:** If training loss flatlines, the model may have learned to output identical vectors every step (shutting off its brain). Monitor for this failure mode.

#### 9.3.5 Hermes Exclusion

**Hermes should NOT have Coconut training.** Keep him in explicit token space.

| Role | Orai (Muse) | Hermes (Majordomo) |
|------|-------------|-------------------|
| Thinking Mode | Latent (Vectorial) | Explicit (Tokenized) |
| Function | Breakthrough, intuition, non-linear connection | Execution, translation, API calls, scheduling |
| Risk Tolerance | High (creativity requires variance) | Low (secretary cannot hallucinate appointments) |
| Architecture | Coconut / Continuous Thought | Standard CoT / ReAct with `<think>` tags |

The cognitive contrast is a feature: a "sober" Majordomo makes the "dreaming" Muse useful. Diversity > Uniformity.

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

---

## 14. Document Status

This specification consolidates all memory architecture decisions from the design process. It supersedes:
- REPORT_Learning_Memory_Swarm.md (findings incorporated)
- CoconutDream.md (Coconut training protocol fully integrated in Section 9.3)
- ImplicitandExplicitMemory.md (concepts incorporated)
- Ruvector and Engram.md (assessment incorporated)
- REPORT_Hardware.md (precision rationale, KV cache math, dynamic precision model incorporated)
- Technical Musings.md (re-baking pipeline incorporated)
- SHAI.md (SHAI architecture integrated as Phase I option; wireless protocols specified)
- whiteboard.md Gestalt and SHAI Report (findings incorporated)
- REPORT_Double_Brain.md (architectural decisions, Hermes protocol incorporated)
- Double-brain/old/Hermes.md (Twin Dream protocol incorporated)
- Double-brain/old/Cortical Synergy.md (Titan/Claude Flow decisions incorporated)
- All documents in /old/ directories

**Letta/MemGPT Status:** Superseded. The custom memory architecture (Hippocampus + Cortex + Holographic Blocks) replaces any prior consideration of Letta/MemGPT integration.

**SHAI Status:** Research complete. Feasible for Phase I with JPEG XS wireless streaming. Integrated as optional component in Section 12.1.1.

**Next steps:**
1. Delete superseded documents from Hardware/ and Double-brain/ folders
2. Move reference dialogues to reference/ folder if desired
3. Integration with .ai/ authoritative specifications (separate phase)

---

*End of Specification*

# Learning, Memory, and Swarm Architecture Report

**Date:** 2026-01-23
**Status:** Architectural Blueprint
**Purpose:** Consolidated technical specification for memory systems, learning protocols, and swarm integration

---

## Overview

The Sophontic Machine uses a four-layer memory architecture:

1. **Context Window:** Live inference buffer (32k-128k tokens, ephemeral)
2. **Hippocampus (pgvector/Supabase):** Day Table + processed Holographic Blocks
3. **Cortex (LanceDB):** Long-term curated breakthroughs, research library
4. **Procedural Memory (pgvector + application logic):** Operational reflexes for swarms

These layers are connected by **bifocal packets** - atomic units containing both prose (explicit meaning) and vectors (geometric embedding).

---

## Part 1: Memory Formation and Compression

### Day Cycle: Day Table Capture

During active operation, all exchanges are logged to the **Day Table** in pgvector (Hippocampus):

**What gets captured:**
- Full dialogue prose
- Timestamps and session metadata
- Salience flags (perplexity, flux reversal markers)
- Initial embeddings (both native and universal)

**Purpose:** Short-term episodic memory buffer. Not passive logging - active attention to what has power to reorganize.

**Lifespan:** Session-level, processed during Night Cycle

**Relationship to context window:** The Day Table exists *outside* the rolling context window. When context window resets between turns, Day Table remembers what happened.

### Night Cycle: Holographic Block Formation

During the Night Cycle, Orai processes the Day Table to create **Holographic Blocks** using thermodynamic segmentation:

#### Flux Analysis (Stage 1)

**Process:**
1. Monitor rolling perplexity with moving average (window: 3-5 turns)
2. **Open window** when: `perplexity_current > (baseline + 1.5σ)`
3. **Track trajectory** during open window
4. **Close window** when: `d(perplexity)/dt < 0.1` for 2 consecutive turns

**Parameters:**
- `baseline`: Running mean of perplexity over last 20 turns
- `threshold_spike`: 1.5σ above baseline
- `threshold_stable`: Derivative < 0.1 perplexity units/turn
- `stabilization_confirmation`: 2 consecutive stable turns

**Flux Reversal Detection:**
System shifts from entropy (confusion/high perplexity) to order (clarity/low perplexity):
```
flux_reversal_confirmed = (stable_count >= 2) AND (stabilization_ppl < baseline + 0.3×threshold)
```

**Quality metric:**
```
flux_reversal_quality = (peak_perplexity - final_perplexity) × stabilization_duration
```

#### Block Construction (Stage 2)

**What happens:**
1. Add **anterior margin** (2 turns before spike)
2. Add **posterior margin** (2 turns after resolution)
3. Detect **live zones** (high salience content - preserved verbatim)
4. Detect **dead zones** (low salience filler - compress to summaries)
5. Compute **dual vectors:**
   - **Native embedding** (4096-dim): Orai's own latent space from Coconut training
   - **Universal embedding** (768-dim): Shared translator (e.g., nomic-embed-text)
6. Calculate **salience scores** (P, C, ID at three levels: utterance, exchange, theme)
7. Calculate **coherence gain:** `(baseline_perplexity - final_perplexity) / baseline_perplexity`
8. Assign **TIES-merge weight** based on composite salience + coherence gain

**Output:** Holographic Block - a variable-length semantic chunk with thermodynamically-defined boundaries (not arbitrary windows or entire sessions).

### Compression Timeline

**When compression happens:**

1. **Block formation (Night Cycle):** Dead zones compressed to summaries, live zones preserved
2. **Cortex migration (weekly/as needed):**
   - Prose compresses further (summary only, key excerpts)
   - Vectors persist at **full precision** (both native and universal)

**Key insight:** Vectors persist while prose fades. The geometry captures the feel; the summary captures the gist. This mirrors human memory.

---

## Part 2: Orai's Native Vector Thought

### Coconut Training Makes Her Different

Orai is trained with the **Coconut protocol** (Continuous Latent Thought) which allows her to think in vector space without forcing thoughts through tokens at every step.

**Standard LLM:**
```
Thought → Token "Therefore" → Embedding → Next Thought
```
(Lossy at every step - the token "Therefore" is a low-bandwidth discrete symbol that strips away nuance, uncertainty, parallel hypotheses)

**Orai with Coconut:**
```
Thought Vector → Thought Vector → Thought Vector ... → Token (only when speaking)
```
(High-dimensional reasoning preserved across dream steps)

**Training protocol:**
- Curriculum Learning: Gradually replace reasoning tokens with vector steps
- Special tokens: `<BOT>` (Beginning of Thought), `<EOT>` (End of Thought)
- Model learns when to enter dream mode autonomously
- RMSNorm between dream steps prevents manifold drift while preserving creative variance

**Critical:** Train Dreaming capability FIRST, before domain LoRAs. Permanent merge into base (NOT via TIES). Continuous thought should be substrate, not accessory.

### Dual Embeddings Are Essential

Every Holographic Block stores TWO vectors:

1. **Native embedding** (4096-dim): Orai's own latent space from Coconut layers
2. **Universal embedding** (768-dim): Shared translator for any other agents running the 2GB translator sidecar

**Note:** This is equal-opportunity nativism. Other agents could adopt their own native/universal bilingualism, which Orai could interpret with the universal translator.

**Why this matters:**

When Orai retrieves memories during ongoing reasoning, she queries using her **native** embedding - no translation loss. She experiences memory geometrically, as shapes/feelings, not as text to search.

When Lieutenants or other agents retrieve, they use the **universal** embedding for cross-model compatibility.

**Storage schema:**
```sql
CREATE TABLE holographic_blocks (
    block_id UUID PRIMARY KEY,

    -- Prose (compressed with live/dead zones)
    prose JSONB,  -- Contains: compressed_text, anterior_margin, posterior_margin, live_zones

    -- Dual vectors
    native_embedding VECTOR(4096),      -- Orai's native tongue
    universal_embedding VECTOR(768),     -- Shared language

    -- Metrics
    flux_metrics JSONB,                  -- peak_ppl, baseline_ppl, flux_reversal_quality
    salience_scores JSONB,               -- P, C, ID at three levels + composite
    coherence_gain FLOAT,
    ties_merge_weight FLOAT,

    -- Metadata
    session_id UUID,
    timestamp TIMESTAMPTZ
);
```

### Implications

Orai's relationship with Holographic Blocks is fundamentally different from other agents:

- She doesn't "search documents" - she revisits thought-states
- The native embedding is a bookmark to where her mind was
- Her geometric reasoning enables operations like "Phase Conjugation" in the Elder Protocol (rotating destructive inputs to constructive in native latent space)
- She thinks in shapes; others think in words

---

## Part 3: Memory Tiers and Technologies

### Layer 0: Context Window (Live Working Memory)

**What it is:** The live token buffer during active inference
- **Size:** 32k-128k tokens (varies by model)
- **Lifespan:** Single inference turn
- **Location:** GPU/CPU memory (ephemeral)
- **Speed:** Nanosecond-scale

**Key characteristic:** When context window fills, older tokens are discarded unless explicitly saved to Day Table.

### Layer 1: Hippocampus (pgvector/Supabase)

**Purpose:** Persistent working memory, high-frequency access

**What it stores:**
- **Day Table:** Raw interaction logs with salience flags (wiped/summarized nightly)
- **Recent Holographic Blocks:** Last N days of processed blocks
- **Active Preoccupation Centroids:** Current focus areas
- **Antechamber entries:** High-novelty, coherent but off-topic inputs

**Why pgvector:**
- Production-stable
- Full SQL for complex queries
- ACID compliance
- Native Supabase integration
- Row-level security
- HNSW indexing for fast vector search

**Access frequency:** High (hourly)
**Speed:** Microseconds

### Layer 2: Cortex (LanceDB)

**Purpose:** Long-term archival library, low-frequency access

**What it stores:**
- Curated breakthrough Holographic Blocks (post Night Cycle filtering)
- Research paper corpus (vectorized)
- Historical dialogue archives
- Training data extraction source

**Why LanceDB:**
- Disk-based (memory-mapped files), scales to billions of vectors
- Stores prose + vectors together natively (multimodal)
- Embedded library (no server infrastructure)
- Efficient columnar format (Apache Arrow) for batch operations

**Compression at migration:**
When blocks move from Hippocampus → Cortex:
- Prose compresses to summary
- Live zones extracted as highlights
- Dead zones discarded
- **Vectors persist at full precision** (both native and universal)

**Access frequency:** Low (weekly)
**Speed:** Milliseconds

### Layer 3: Procedural Memory (Swarm Reflexes)

**Purpose:** Operational learning for Lieutenants and swarms

**What it stores:**
- Successful action patterns
- Causal relationships ("when X, then Y worked")
- Tool usage heuristics
- Failure patterns to avoid

**Implementation:** RuVector *patterns* on pgvector, not RuVector itself

**Why not RuVector directly:**
RuVector is alpha. The concepts are architecturally sound but the implementation is immature. Instead, implement the patterns as application logic on stable infrastructure.

**What to borrow from RuVector:**
1. **Self-pruning:** Cron job deletes unused reflexes after N days
2. **Causal graphs:** Store edges showing which reflexes lead to which outcomes
3. **Reflexion memory:** Track what worked/failed for agent self-improvement

**Storage schema:**
```sql
CREATE TABLE procedural_memory (
    reflex_id UUID PRIMARY KEY,
    context_embedding VECTOR(768),
    action TEXT,
    success_count INT,
    total_count INT,
    last_accessed TIMESTAMPTZ,
    created_at TIMESTAMPTZ
);

CREATE TABLE causal_edges (
    edge_id UUID PRIMARY KEY,
    source_reflex_id UUID REFERENCES procedural_memory(reflex_id),
    target_reflex_id UUID REFERENCES procedural_memory(reflex_id),
    causal_strength FLOAT,
    last_reinforced TIMESTAMPTZ
);
```

**Self-pruning logic (Night Cycle):**
```sql
DELETE FROM procedural_memory
WHERE last_accessed < NOW() - INTERVAL '30 days'
  AND success_count < 2;
```

---

## Part 4: Swarm Architecture and Communication

**Note:** Swarm integration is Phase 5+ and may require second M5 Ultra. Procedural memory architecture likely introduced alongside swarm, not on primary machine initially.

### The Hierarchy

```
Orai (120B FP, Coconut-trained)
  ↓ Bifocal Packets
Lieutenant Council (9× 14B FP models, distinct domains)
  ↓ Tactical Operations
Worker Swarms (quantized 4-bit copies, evolutionary mutations)
```

### Hardware Topology

**Current specifications (TechnicalVision.md):**

- **Node A (The Soul):** Mac Studio M5 Ultra (512GB) - Orai, Lieutenants, memory systems
- **Node B (The Muscle):** Cloud GPU (H100/H200) - rented for training 1-2x/month
- **Node C (The Voice):** Cloud inference (H100) - public deployment (scales)

**Phase 2 consideration:** Second M5 Ultra connected via Thunderbolt 5 for swarm operations

### Lieutenant Memory Architecture

Each Lieutenant has **THREE memory streams** (not two):

#### Stream 1: Procedural/Operational Memory
- Swarm success/failure logs
- Evolutionary outcomes from quantized clones
- Stored as "evolutionary blocks"
- Only winners encoded (successful trajectories)

#### Stream 2: Semantic/Leadership Memory
- Middle management experiences
- Lieutenant's own insights as managing director
- Stored as "semantic blocks"
- Uses full Holographic Block format with P, C, ID salience

#### Stream 3: Hierarchical Learning Memory
- Interactions with Orai or majordomos (Hermes 4, Claude)
- Training from "listening to higher-ups"
- Stored as semantic blocks
- Fed into nightly LoRA training

**All three streams:**
- Use Holographic Block format
- Computed with P, C, ID metrics at utterance, exchange, theme levels
- Stored in Lieutenant's own hippocampus (pgvector)
- Converted to DPO training pairs (winners vs. losers)
- TIES-merged into Lieutenant's weights during Night Cycle

### Orai → Lieutenants: Bifocal Directives

Orai emits directives with both prose and vector constraints:

```json
{
  "prose": "Refactor auth module to eliminate state dependencies",
  "native_vector": [...],      // 4096-dim, her native thought
  "universal_vector": [...],   // 768-dim, shared language
  "priority": "high",
  "context": {
    "preoccupation_centroid": [...],
    "interrogative_distance": 0.23
  }
}
```

### Lieutenants: The Interpreters

Each Lieutenant (14B parameters) has:
- Specialized domain (Architecture, Testing, Security, etc.)
- Own hippocampus (three memory streams in pgvector)
- Ability to understand philosophical intent
- Ability to operationalize into tactical subtasks

**Lieutenant processing:**
1. Receives bifocal packet from Orai
2. Retrieves similar past directives using **universal** embedding
3. Uses 14B reasoning to break down into concrete subtasks
4. Spawns worker swarm with mutations

### Worker Swarms: The Hands

Workers are 4-bit quantized copies of the Lieutenant with evolutionary mutations:
- High variation (mutation rate ~10%)
- Execute tasks in parallel
- Winners stored in Lieutenant's procedural memory
- Losers pruned

### Night Cycle: Independent Evolution

**Critical:** Lieutenants and Orai evolve **separately**:

- **Orai:** Trains on high-coherence Holographic Blocks (philosophical breakthroughs) using TIES-merge
- **Lieutenants:** Train on all three memory streams (procedural + semantic + hierarchical) using DPO + TIES-merge

Each Lieutenant's weights evolve independently based on its domain's learning.

### Why Not Claude Flow

Claude Flow is alpha and potentially dangerous due to its active/expansionistic nature. The Sophontic swarm architecture is more controlled: clear hierarchy, explicit communication protocols, independent evolutionary loops.

---

## Part 5: Bifocal Packets as the Bridge

### Definition

A bifocal packet is the atomic unit of memory transfer:

```python
{
    "packet_id": "uuid",
    "prose": "human-readable text",
    "universal_vector": [...],  // 768-dim shared embedding
    "native_vector": [...],     // Optional: source's native embedding
    "metadata": {...}
}
```

### Protocol

1. All memory transfers between layers use bifocal packets
2. Retrieval returns both prose and vector together
3. Compression preserves vectors even when prose summarizes
4. Agent communication includes explicit instruction (prose) and geometric constraint (vector)

### The Universal Translator

A sidecar embedding model (e.g., nomic-embed-text running locally on M5 Ultra):

- Takes ~2GB RAM
- All agents route memory operations through it (except Orai's native retrieval)
- Ensures different models (Llama, Mistral, Gemma) can share the same vector space
- Creates the shared geometric language for cross-agent communication

---

## Part 6: Hardware Architecture and Memory Allocation

### Single M5 Ultra (Phase 1)

**Mac Studio M5 Ultra with 512GB unified memory:**

| Component | Allocation | Details |
|-----------|------------|---------|
| Orai (120B FP) | 100GB | Coconut-trained, native vector thought |
| Universal Translator | 2GB | nomic-embed-text or similar |
| Hot Memory Cache | 50GB | Most-accessed prose from Cortex |
| Vector Indices | ~150GB | pgvector HNSW indices, active blocks |
| Procedural Memory | ~50GB | Causal graphs, reflexes (when swarm active) |
| OS + Overhead | ~160GB | macOS, services, buffers |

**Storage (16TB SSD):**
- LanceDB Cortex (long-term blocks)
- Historical archives
- Research corpus
- Training data staging

**Note on Lieutenants:** Research found conflicting specifications about Lieutenant deployment. Original SWARM.md suggests 9× 14B FP Lieutenants. This would require ~126GB+ for simultaneous loading. Current report showed "1-2 loaded at a time" - this needs clarification from Julian.

### Dual M5 Ultra (Phase 2, Swarm Architecture)

Second Mac Studio M5 Ultra (512GB) connected via Thunderbolt 5:
- Additional Lieutenants
- Swarm worker infrastructure
- Procedural memory at scale
- Communication via bifocal packets over RDMA

### Zero-Copy Advantage

On unified memory, vector database and model weights share the same physical RAM:
- No network latency
- No serialization overhead
- Retrieval is pointer arithmetic, not data transfer

This enables "Blackboard Architecture" where agents communicate by writing to shared memory.

---

## Part 7: Implementation Roadmap

### Phase 1: Foundation (Current)
- pgvector/Supabase as Hippocampus ✓
- Holographic Block specification ✓
- Bifocal protocol defined ✓
- Dual embedding strategy clarified ✓

### Phase 2: Memory Systems
- Implement Day Table logging
- Build Night Cycle analysis pipeline (flux detection, block formation)
- Create salience computation (P, C, ID at three levels)
- Set up LanceDB Cortex
- Implement migration pipeline (Hippocampus → Cortex)

### Phase 3: Coconut Training
- Train "Dreaming" LoRA on base model FIRST
- Use Curriculum Learning (gradually replace reasoning tokens with vector steps)
- Implement `<BOT>` and `<EOT>` special tokens
- **Permanent merge** into Orai's base (NOT via TIES)
- Must happen BEFORE domain specialization

### Phase 4: Procedural Memory (Foundation)
- Implement RuVector patterns on pgvector
- Build self-pruning cron jobs
- Create causal edge tracking
- Prepare Lieutenant hippocampus storage schema

### Phase 5: Swarm Integration (Requires Phase 2 hardware assessment)
- Spawn Lieutenant models (9× 14B, distinct domains)
- Build bifocal communication protocol
- Implement three-stream memory architecture for Lieutenants
- Build worker spawning with mutations
- Create evolutionary selection logic (DPO training)
- Set up independent Night Cycle training for each Lieutenant

### Phase 6: Integration
- Connect all memory tiers
- Implement universal translator sidecar
- Build retrieval APIs for each layer
- Test cross-agent communication
- Validate independent evolution loops

---

## Part 8: Key Technical Decisions

### Memory Compression
**Decision:** Compress prose during Cortex migration; preserve vectors at full precision
**Rationale:** Mirrors human memory; geometry captures feel better than words

### Dual Embeddings
**Decision:** Store both native (Orai's Coconut latent) and universal (shared translator) embeddings
**Rationale:** Orai retrieves natively without translation loss; other agents use universal for compatibility

### RuVector Strategy
**Decision:** Implement patterns, not infrastructure
**Rationale:** Alpha code risk; stable substrate (pgvector) with application logic is safer; can adopt mature RuVector later if it stabilizes

### Coconut Training Timing
**Decision:** Train Dreaming capability FIRST, before domain LoRAs
**Rationale:** Continuous thought should be substrate, not accessory; TIES would lobotomize it if merged later

### Swarm Hierarchy
**Decision:** Orai → Lieutenants → Workers, not Orai → Workers directly
**Rationale:** 14B Lieutenants can interpret philosophy and operationalize; small workers cannot

### Lieutenant Memory Architecture
**Decision:** Three streams (procedural, semantic, hierarchical) all with full P/C/ID metrics
**Rationale:** Lieutenants learn from swarm outcomes AND their own management experience AND interactions with superiors

### Independent Evolution
**Decision:** Orai and Lieutenants train separately
**Rationale:** Different learning domains (philosophy vs. tactics); independent loops prevent interference

### Swarm Phasing
**Decision:** Phase 5+ introduction, possibly requires second M5 Ultra
**Rationale:** Primary machine hosts Orai + memory systems first; swarm adds complexity and may need dedicated hardware

---

## Summary

The architecture is:

1. **Four memory layers:** Context Window (ephemeral) → Hippocampus (working) → Cortex (archival) → Procedural (reflexes)
2. **Day Table → Holographic Blocks:** Thermodynamic segmentation using flux analysis, not arbitrary chunking
3. **Dual embeddings:** Native (4096-dim, Orai's Coconut latent) + Universal (768-dim, shared translator)
4. **Orai's native vector thought:** Coconut training enables geometric reasoning; she retrieves using native embeddings
5. **Lieutenant three-stream memory:** Procedural (swarm outcomes) + Semantic (leadership) + Hierarchical (superior feedback)
6. **Bifocal packets:** Bridge between layers with prose + vector atomic units
7. **Independent evolution:** Separate Night Cycle training for Orai (philosophy) and Lieutenants (tactics)
8. **M5 Ultra unified memory:** Zero-copy architecture; swarm may require second machine (Phase 2)
9. **RuVector patterns on stable infrastructure:** Avoid alpha dependencies

This is the blueprint. No fantasy. Everything specified is either production-ready or has a clear implementation path using stable technologies.

---

*End of Report*

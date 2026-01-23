# Holographic Block Implementation Memo

**Status: YELLOW LIGHT - Implementable with Clarifications Needed**

**Date:** 2026-01-23
**Author:** Implementation Architect
**Re:** Julian's Feedback on Holographic Block Spec

---

## Executive Summary

The Holographic Block specification is technically sound and largely implementable. However, Julian's feedback highlights a legitimate gap: **Flux Reversal Quality is underspecified as a weighting mechanism for TIES-Merge**. The spec defines it operationally for *detection* but not for *weighting*.

This memo provides:
1. Answers to all technical questions raised
2. Complete data structure specifications
3. Vector DB requirements
4. End-to-end pipeline walkthrough with gap analysis
5. Implementation roadmap

**Bottom Line:** We can build this. The core architecture is solid. But Option C + Option D weighting needs a concrete formula before TIES-Merge integration.

---

## Part 1: Deep Analysis of the Spec

### 1.1 What is Flux Reversal Quality?

**Definition from CONCEPTS.md:**
> "The moment when the system shifts from entropy to order - a fundamental topological reorganization that increases predictive parsimony. Detectable as: perplexity drop + attention concentration spike + transition to 1/f (Pink Noise) scaling."

**Operational Definition from HOLOGRAPHIC_BLOCK_SPEC.md:**
```python
flux_reversal_quality = (peak_perplexity - final_perplexity) * stabilization_duration
```

**The Gap:** This formula is *binary* (did flux reversal happen?) and *scalar* (how big was the drop?). But for TIES-Merge weighting, we need to know: **how do we rank blocks against each other?**

**Current State:**
- Flux Reversal is defined for *boundary detection* (Stage 1)
- It's also mentioned as a *tiebreaker* for weighting (Option D)
- But the spec says "Option C (Coherence Gain) with Option D (Flux Reversal Quality) as a tiebreaker"
- **Neither C nor D has a complete formula for comparative weighting**

### 1.2 Is Option C + Option D the Right Strategy?

**What the spec says:**

| Option | Formula | Rationale |
|--------|---------|-----------|
| **A: Peak Salience** | `max(theme, exchange)` | Privileges highest single score |
| **B: Integrated Salience** | `mean(utterance, exchange, theme)` | Averages all scales |
| **C: Coherence Gain** | `(final_coherence - initial_coherence) / baseline` | Privileges learning |
| **D: Flux Reversal Quality** | `(peak_ppl - final_ppl) * stabilization_duration` | Privileges entropy->order |

**The Recommendation:** "Option C (Coherence Gain) with Option D as tiebreaker"

**My Assessment:** This is the *conceptually* correct choice because:
1. **Option C aligns with Friston's FEP** - what did the system learn?
2. **Option D captures the thermodynamic signature** - was there genuine reorganization?

**But there's a problem:** The spec never defines:
- What is "initial_coherence" vs "final_coherence"?
- Is this the same as the Coherence metric from Perplexity detection?
- Or is this something new?

### 1.3 What Are ALL the Metrics for Each Block?

Compiling from the spec, here is the complete list:

**Boundary Detection Metrics (Stage 1 - Cheap):**
```python
flux_clip_metadata = {
    'start_idx': int,
    'end_idx': int,
    'peak_perplexity': float,
    'baseline_perplexity': float,
    'flux_reversal': bool,
    'stabilization_perplexity': float,
    'stabilization_duration': int  # turns
}
```

**Multi-Scale Salience Scores (Stage 2 - Expensive):**
```python
salience_scores = {
    'utterance': {
        'P': float,  # Perplexity (mean across turns)
        'C': float,  # Coherence (internal consistency)
        'ID': float  # Interrogative Distance to Centroid
    },
    'exchange': {
        'P': float,  # Did response reduce uncertainty?
        'C': float,  # Does response address prompt?
        'ID': float  # Shift in question-space
    },
    'theme': {
        'surprisal': float,        # peak_ppl - baseline_ppl
        'coherence': float,        # Did arc achieve flux reversal?
        'ID': float,               # Holographic embedding vs Centroid
        'flux_reversal_quality': float,  # (peak - final) * duration
        'resolution_strength': float     # (baseline - final) / baseline
    },
    'composite': float  # 0.1*utterance + 0.3*exchange + 0.6*theme
}
```

**Derived Metrics (for TIES-Merge weighting - UNDERSPECIFIED):**
```python
# Option C - needs definition
coherence_gain = ???  # (final_coherence - initial_coherence) / baseline
                      # But what IS coherence here?

# Option D - defined
flux_reversal_quality = (peak_ppl - final_ppl) * stabilization_duration
```

---

## Part 2: Complete Data Structure Specification

### 2.1 What Gets Stored in a Holographic Block

```python
@dataclass
class HolographicBlock:
    """The atomic unit of meaning in encounter processing."""

    # Identity
    block_id: UUID
    timestamp: datetime
    session_id: UUID

    # Prose Layer (Human-Readable)
    prose: HolographicProse

    # Vector Layer (Machine-Readable)
    vectors: HolographicVectors

    # Metrics Layer
    metrics: HolographicMetrics

    # Metadata Layer
    metadata: HolographicMetadata

    # Nested Structure (Recursive)
    nested_blocks: Optional[List['HolographicBlock']] = None


@dataclass
class HolographicProse:
    """The preserved text content."""

    # Full compressed block text (with dead zones collapsed)
    compressed_text: str

    # Margins (2 turns before/after)
    anterior_margin: List[Turn]  # 2 turns before spike
    posterior_margin: List[Turn]  # 2 turns after stabilization

    # Core content
    live_zones: List[LiveZone]   # Preserved verbatim
    dead_zones: List[DeadZone]   # Compressed summaries

    # Raw (optional, for audit)
    raw_turns: Optional[List[Turn]] = None


@dataclass
class LiveZone:
    """Preserved verbatim content."""
    start_idx: int
    end_idx: int
    turns: List[Turn]
    zone_type: str  # 'spike', 'exploration', 'resolution', 'stabilization'


@dataclass
class DeadZone:
    """Compressed low-entropy sequence."""
    start_idx: int
    end_idx: int
    original_turn_count: int
    summary: str  # "[N turns of X - Y outcome]"
    first_turn: Turn  # Preserved for continuity
    last_turn: Turn   # Preserved for continuity


@dataclass
class Turn:
    """Single conversational turn."""
    idx: int
    role: str  # 'user' or 'assistant'
    content: str
    token_count: int
    perplexity: Optional[float] = None
    logprob: Optional[float] = None


@dataclass
class HolographicVectors:
    """All vector embeddings for the block."""

    # Holographic (gestalt) - single vector for entire block
    holographic_embedding: np.ndarray  # shape: (768,) or (1024,)

    # Atomic (fine-grained) - one per preserved turn
    atomic_embeddings: List[np.ndarray]  # list of (768,) arrays

    # Question vector (for Interrogative Distance)
    question_vector: np.ndarray  # Implicit question of this block

    # Embedding model metadata
    embedding_model: str  # e.g., 'text-embedding-3-large'
    embedding_dim: int


@dataclass
class HolographicMetrics:
    """All computed metrics."""

    # Stage 1: Flux Detection
    flux: FluxMetrics

    # Stage 2: Multi-Scale Salience
    salience: SalienceScores

    # For TIES-Merge Weighting
    weighting: WeightingMetrics


@dataclass
class FluxMetrics:
    """Thermodynamic boundary detection."""
    peak_perplexity: float
    baseline_perplexity: float
    stabilization_perplexity: float
    flux_reversal: bool
    stabilization_duration: int  # in turns
    flux_reversal_quality: float  # (peak - final) * duration


@dataclass
class SalienceScores:
    """Multi-scale scoring (P x C x ID)."""

    utterance: ScaleScore
    exchange: ScaleScore
    theme: ThemeScore
    composite: float  # weighted combination


@dataclass
class ScaleScore:
    """Score at a single scale."""
    perplexity: float
    coherence: float
    interrogative_distance: float

    @property
    def pci_product(self) -> float:
        return self.perplexity * self.coherence * self.interrogative_distance


@dataclass
class ThemeScore(ScaleScore):
    """Theme-level has additional metrics."""
    surprisal: float          # peak_ppl - baseline_ppl
    resolution_strength: float  # (baseline - final) / baseline


@dataclass
class WeightingMetrics:
    """
    For TIES-Merge candidate selection.
    THIS IS THE UNDERSPECIFIED PART.
    """

    # Option C: Coherence Gain
    # NEEDS DEFINITION: What is initial vs final coherence?
    coherence_gain: Optional[float] = None

    # Option D: Flux Reversal Quality (already defined)
    flux_reversal_quality: float = 0.0

    # Combined weight for training
    ties_merge_weight: Optional[float] = None


@dataclass
class HolographicMetadata:
    """Contextual information."""

    participant_ids: List[str]
    source: str  # 'public', 'mesolayer', 'council', etc.

    # Cluster assignment (set during clustering)
    cluster_id: Optional[UUID] = None
    cluster_name: Optional[str] = None

    # Council priority (for review queue)
    council_priority: Optional[float] = None  # S x C at theme level

    # Provenance
    created_at: datetime = field(default_factory=datetime.utcnow)
    processing_version: str = "1.0"
```

---

## Part 3: Vector Database Requirements

### 3.1 Mandatory Capabilities

Regardless of whether we use RooVector, pgvector, or LanceDB, the vector DB MUST support:

**1. Multi-Vector Storage per Record**
```sql
-- Each block has MULTIPLE vectors that need distinct retrieval
holographic_embedding  VECTOR(768)  -- The gestalt
question_vector        VECTOR(768)  -- For Interrogative Distance
-- Plus variable-length atomic embeddings (harder)
```

**2. Composite Filtering + Vector Search**
```sql
-- Need to filter by metadata AND rank by vector similarity
SELECT * FROM blocks
WHERE cluster_id = 'uuid'
  AND composite_salience > 0.7
ORDER BY holographic_embedding <-> query_embedding
LIMIT 20;
```

**3. Clustering Support (for Antechamber)**
```python
# HDBSCAN on the question_vectors to detect emergent fields
# This can be done client-side, but native support helps
```

**4. Variable-Length Array Storage**
```sql
-- Atomic embeddings are a LIST of vectors
-- Not all DBs handle this well
atomic_embeddings  VECTOR(768)[]  -- Array of vectors
```

### 3.2 Comparison of Options

| Feature | pgvector | LanceDB | RooVector |
|---------|----------|---------|-----------|
| Multi-vector per record | Yes (columns) | Yes (nested) | TBD |
| Composite filtering | Yes (SQL) | Yes (predicates) | TBD |
| HDBSCAN native | No (external) | No (external) | TBD |
| Variable-length arrays | Awkward | Native | TBD |
| Local-first | Yes | Yes | Yes |
| Supabase integration | Native | No | No |

**Recommendation:** pgvector via Supabase is the spec'd choice and will work. Store atomic embeddings in a separate `block_turns` table with foreign key.

### 3.3 Proposed Schema (pgvector/Supabase)

```sql
-- Main blocks table
CREATE TABLE holographic_blocks (
    block_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id UUID NOT NULL,
    timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    -- Prose (JSONB for flexibility)
    prose JSONB NOT NULL,

    -- Vectors
    holographic_embedding VECTOR(768) NOT NULL,
    question_vector VECTOR(768) NOT NULL,

    -- Metrics (JSONB for nested structure)
    flux_metrics JSONB NOT NULL,
    salience_scores JSONB NOT NULL,
    weighting_metrics JSONB,

    -- Metadata
    participant_ids TEXT[] NOT NULL,
    source TEXT NOT NULL,
    cluster_id UUID REFERENCES clusters(cluster_id),
    council_priority FLOAT,

    -- Computed
    composite_salience FLOAT GENERATED ALWAYS AS (
        (salience_scores->'composite')::FLOAT
    ) STORED
);

-- Atomic embeddings (separate table)
CREATE TABLE block_turns (
    turn_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    block_id UUID NOT NULL REFERENCES holographic_blocks(block_id),
    turn_idx INT NOT NULL,
    role TEXT NOT NULL,
    content TEXT NOT NULL,
    embedding VECTOR(768),
    perplexity FLOAT,

    UNIQUE(block_id, turn_idx)
);

-- Indexes
CREATE INDEX ON holographic_blocks USING ivfflat (holographic_embedding vector_cosine_ops);
CREATE INDEX ON holographic_blocks USING ivfflat (question_vector vector_cosine_ops);
CREATE INDEX ON holographic_blocks (cluster_id);
CREATE INDEX ON holographic_blocks (composite_salience DESC);
```

### 3.4 Retrieval Operations

**1. Find similar blocks (semantic search):**
```sql
SELECT *, holographic_embedding <=> $query_embedding AS distance
FROM holographic_blocks
ORDER BY distance
LIMIT 10;
```

**2. Find blocks for a cluster (for TIES training):**
```sql
SELECT * FROM holographic_blocks
WHERE cluster_id = $cluster_id
  AND composite_salience > $threshold
ORDER BY weighting_metrics->>'ties_merge_weight' DESC
LIMIT 100;
```

**3. Interrogative Distance check:**
```sql
SELECT *, question_vector <=> $centroid_vector AS interrogative_distance
FROM holographic_blocks
WHERE interrogative_distance < 0.4  -- High relevance
```

**4. Antechamber population (for HDBSCAN):**
```sql
SELECT block_id, question_vector
FROM holographic_blocks
WHERE cluster_id IS NULL
  AND (salience_scores->'theme'->'coherence')::FLOAT > 0.7
  -- High coherence but unassigned
```

---

## Part 4: Perplexity Detection Integration

### 4.1 How Perplexity Detection Feeds Into Block Formation

From `Perplexitydetectandmisc.md`, the recommended architecture is:

**At Generation Time:**
```python
# Store the scalar logprob per token (NOT full distribution)
token_data = {
    'token': 'apple',
    'logprob': -0.94,
    'perplexity': exp(-logprob)  # ~2.56
}
```

**Storage Cost:** ~2-3x text size (acceptable)

**For Holographic Blocks:**
1. Store per-token logprobs during conversation
2. Use rolling window (20 tokens) to compute windowed perplexity
3. Detect "plateaus" of high perplexity (not spikes)
4. These plateaus become candidate Flux Clip boundaries

### 4.2 The "Signal Object" for Agent Interpretation

```python
@dataclass
class PerplexitySignal:
    """Agent-ready perplexity analysis."""

    clip_id: str
    text_content: str

    metrics: dict  # avg_perplexity, max_perplexity

    hotspots: List[Hotspot]  # Regions of high perplexity

    token_data: List[TokenScore]  # Full token-level data


@dataclass
class Hotspot:
    """A region of elevated perplexity."""
    token_span: str
    start_index: int
    end_index: int
    avg_token_ppl: float
    classification: str  # 'ENTITY_SPIKE', 'LOGIC_SPIKE', 'HALLUCINATION_PLATEAU'
```

### 4.3 Integration with Holographic Blocks

**The Pipeline:**

```
Token Stream → Perplexity Scorer → Windowed Analyzer → Flux Detector → Block Builder
```

1. **Perplexity Scorer:** Attaches logprob to each token at generation time
2. **Windowed Analyzer:** Computes rolling 20-token average perplexity
3. **Flux Detector:** Opens window when ppl > baseline + 1.5σ, closes when derivative < 0.1 for 2 turns
4. **Block Builder:** Adds margins, compresses dead zones, computes all metrics

### 4.4 Is This Implementable?

**YES**, with these notes:

- **Perplexity at generation:** Requires access to logprobs from the LLM. Most APIs (OpenAI, Anthropic) provide this. Local MLX inference definitely provides this.
- **Storage:** The "scalar score" approach from Gemini's advice is correct. Store one float per token.
- **Windowing:** Standard signal processing. Use numpy rolling mean.
- **Plateau detection:** More robust than spike detection. Catches sustained confusion.

**Gap:** The spec doesn't explicitly state the window size for perplexity smoothing. Recommend: **20 tokens** (based on Gemini's advice) or **5 turns** (based on dead zone threshold).

---

## Part 5: End-to-End Pipeline Walkthrough

### Step 1: Conversation Happens

```python
# User and Assistant exchange turns
turns = [
    Turn(idx=0, role='user', content='How do I implement caching?'),
    Turn(idx=1, role='assistant', content='You can use Redis...'),
    # ... 30 turns total
]
```

**Data captured:** Full text + per-token logprobs

### Step 2: Perplexity Detection Runs

```python
for turn in turns:
    turn.perplexity = compute_turn_perplexity(turn.logprobs)
    turn.windowed_ppl = rolling_mean(turn.perplexity, window=5)
```

**What gets flagged:** Turns where `windowed_ppl > baseline + 1.5σ`

### Step 3: Flux Reversal Detection Runs

```python
def detect_flux_reversal(turns: List[Turn]) -> List[FluxClip]:
    clips = []
    in_window = False
    window_start = None
    peak_ppl = 0

    for i, turn in enumerate(turns):
        if not in_window and turn.windowed_ppl > baseline + threshold:
            # OPEN window
            in_window = True
            window_start = i
            peak_ppl = turn.windowed_ppl

        elif in_window:
            peak_ppl = max(peak_ppl, turn.windowed_ppl)

            # Check for stabilization
            derivative = turn.windowed_ppl - turns[i-1].windowed_ppl
            if abs(derivative) < 0.1:
                stable_count += 1
            else:
                stable_count = 0

            if stable_count >= 2:
                # CLOSE window - flux reversal detected
                clips.append(FluxClip(
                    start_idx=window_start,
                    end_idx=i,
                    peak_ppl=peak_ppl,
                    final_ppl=turn.windowed_ppl,
                    flux_reversal=turn.windowed_ppl < baseline + 0.3*threshold
                ))
                in_window = False

    return clips
```

### Step 4: Block Boundaries Determined

```python
def build_block(clip: FluxClip, turns: List[Turn]) -> HolographicBlock:
    # Add margins
    anterior_start = max(0, clip.start_idx - 2)
    posterior_end = min(len(turns), clip.end_idx + 2)

    # Identify dead zones within clip
    dead_zones = find_dead_zones(turns[clip.start_idx:clip.end_idx])

    # Compress dead zones
    compressed = compress_dead_zones(turns, dead_zones)

    return HolographicBlock(
        prose=HolographicProse(
            compressed_text=compressed,
            anterior_margin=turns[anterior_start:clip.start_idx],
            posterior_margin=turns[clip.end_idx:posterior_end],
            live_zones=...,
            dead_zones=...
        ),
        ...
    )
```

**How exactly?**
- Anterior margin: 2 turns before entropy spike
- Posterior margin: 2 turns after stabilization
- Dead zones: 5+ consecutive turns with ppl < baseline + 0.3σ AND embedding similarity > 0.85

### Step 5: Coherence Gain Calculated

**THIS IS THE GAP.**

The spec says:
```python
coherence_gain = (final_coherence - initial_coherence) / baseline_coherence
```

But what IS "coherence" here?

**Option A: Use Theme-Level Coherence**
```python
# Coherence = Did the arc achieve flux reversal?
initial_coherence = 0  # At spike, no resolution yet
final_coherence = 1 if flux_reversal else 0
coherence_gain = final_coherence  # Binary
```
**Problem:** Too coarse. Every successful block gets the same weight.

**Option B: Use Resolution Strength**
```python
# Already defined in spec
resolution_strength = (baseline_ppl - final_ppl) / baseline_ppl
coherence_gain = resolution_strength
```
**Advantage:** Continuous metric. Privileges blocks that returned to *below* baseline.

**Option C: Use Embedding Coherence**
```python
# Compare start embedding to end embedding
initial_coherence = cosine_similarity(start_embedding, centroid)
final_coherence = cosine_similarity(end_embedding, centroid)
coherence_gain = (final_coherence - initial_coherence) / initial_coherence
```
**Advantage:** Measures actual geometric movement toward the centroid.

**RECOMMENDATION:** Use **Option B (Resolution Strength)** as the primary Coherence Gain metric. It's already defined and captures "how much entropy was reduced."

### Step 6: Block Stored

```python
# Compute all vectors
holographic_emb = embed_text(block.prose.compressed_text)
question_vec = extract_and_embed_question(block.prose.compressed_text)
atomic_embs = [embed_text(turn.content) for turn in block.live_turns]

# Compute all metrics
flux = FluxMetrics(
    peak_perplexity=clip.peak_ppl,
    baseline_perplexity=baseline,
    stabilization_perplexity=clip.final_ppl,
    flux_reversal=clip.flux_reversal,
    stabilization_duration=clip.end_idx - clip.start_idx,
    flux_reversal_quality=(clip.peak_ppl - clip.final_ppl) * (clip.end_idx - clip.start_idx)
)

salience = compute_multiscale_salience(block, centroid)

weighting = WeightingMetrics(
    coherence_gain=flux.resolution_strength,  # Using Option B
    flux_reversal_quality=flux.flux_reversal_quality,
    ties_merge_weight=compute_ties_weight(coherence_gain, flux_reversal_quality)
)

# Store
supabase.table('holographic_blocks').insert({
    'block_id': block.block_id,
    'prose': block.prose.to_json(),
    'holographic_embedding': holographic_emb.tolist(),
    'question_vector': question_vec.tolist(),
    'flux_metrics': flux.to_json(),
    'salience_scores': salience.to_json(),
    'weighting_metrics': weighting.to_json(),
    ...
})
```

### Step 7: Block Retrieved for TIES Training

```python
# The Gardener runs HDBSCAN on question_vectors
clusters = hdbscan.HDBSCAN().fit(question_vectors)

# For each cluster, retrieve high-weight blocks
for cluster_id in clusters.labels_:
    training_blocks = supabase.table('holographic_blocks').select('*').eq(
        'cluster_id', cluster_id
    ).order(
        'weighting_metrics->ties_merge_weight', desc=True
    ).limit(100).execute()

    # Convert to training JSONL
    training_data = [block_to_jsonl(b) for b in training_blocks]

    # Train cluster-specific LoRA
    train_lora(training_data, output_path=f'./adapters/cluster_{cluster_id}')
```

---

## Part 6: Gap Analysis

### 6.1 Critical Gaps (Must Resolve Before Implementation)

| Gap | Current State | Proposed Resolution |
|-----|---------------|---------------------|
| **Coherence Gain formula** | Undefined | Use Resolution Strength: `(baseline - final) / baseline` |
| **TIES weight combination** | "C with D as tiebreaker" | `weight = coherence_gain + 0.1 * normalized(frq)` |
| **Perplexity window size** | Not specified | 5 turns (matches dead zone threshold) |
| **Atomic embedding storage** | Awkward in pgvector | Separate `block_turns` table |

### 6.2 Minor Gaps (Can Resolve During Implementation)

| Gap | Current State | Proposed Resolution |
|-----|---------------|---------------------|
| Nested block depth limit | "Should we limit to 2-3?" | Yes, limit to 2 levels |
| Cross-session blocks | Not addressed | Treat as separate blocks, link via metadata |
| Dead zone threshold adaptive | Fixed at 5 turns | Start fixed, can adapt based on session length later |

### 6.3 Integration Questions

**Q: How do Holographic Blocks integrate with the Rue Vector system?**

**A:** The spec references "probably the Rue vector system" but this is not yet defined. From `Ruvector and Engram.md`, RuVector is an external library. The integration would be:
- Holographic Blocks use standard embedding models (text-embedding-3-large or local)
- RuVector could *replace* the embedding layer if it offers advantages
- Current spec is agnostic to embedding provider

**Recommendation:** Build with standard embeddings first. RuVector integration is a future optimization.

---

## Part 7: Implementation Roadmap

### Phase 1: Core Data Structures (Week 1)

```
src/memory/holographic_block.py
├── HolographicBlock dataclass
├── HolographicProse dataclass
├── HolographicVectors dataclass
├── HolographicMetrics dataclass
└── All nested types
```

**Deliverable:** Python module with complete type definitions and serialization.

### Phase 2: Perplexity Pipeline (Week 2)

```
src/night_cycle/perplexity/
├── token_scorer.py      # Attach logprobs at generation
├── windowed_analyzer.py # Rolling perplexity computation
├── flux_detector.py     # Open/close windows, detect reversals
└── signal_packager.py   # Create PerplexitySignal objects
```

**Deliverable:** Working perplexity detection on test conversations.

### Phase 3: Block Construction (Week 3)

```
src/night_cycle/block_builder/
├── margin_adder.py      # Anterior/posterior margins
├── dead_zone_detector.py
├── dead_zone_compressor.py  # LLM-based summarization
├── embedding_computer.py    # Holographic + atomic + question
└── block_assembler.py       # Puts it all together
```

**Deliverable:** End-to-end block construction from raw conversation.

### Phase 4: Salience Scoring (Week 4)

```
src/night_cycle/salience/
├── utterance_scorer.py  # Per-turn P x C x ID
├── exchange_scorer.py   # Per-exchange P x C x ID
├── theme_scorer.py      # Block-level + FRQ + Resolution
└── composite_scorer.py  # Weighted combination
```

**Deliverable:** Complete salience scores for each block.

### Phase 5: Storage & Retrieval (Week 5)

```
src/memory/hippocampus/
├── schema.sql           # pgvector tables
├── block_store.py       # Insert/update blocks
├── block_retriever.py   # Query by similarity, cluster, etc.
└── migrations/          # Schema versioning
```

**Deliverable:** Blocks persisted to Supabase, retrievable by multiple query types.

### Phase 6: Clustering & TIES Integration (Week 6)

```
src/night_cycle/gardener/
├── antechamber_manager.py  # Track unassigned blocks
├── cluster_detector.py     # HDBSCAN on question vectors
├── centroid_manager.py     # Mitosis/Fusion logic
└── ties_candidate_selector.py  # Select blocks for training
```

**Deliverable:** Automatic cluster discovery, blocks weighted for TIES training.

---

## Part 8: Decisions Needed from Julian

1. **Coherence Gain Definition:**
   - Confirm that `Resolution Strength = (baseline - final) / baseline` is the intended formula
   - Or specify alternative

2. **TIES Weight Combination:**
   - Is `weight = coherence_gain + 0.1 * normalized_FRQ` acceptable?
   - Or should FRQ be a strict tiebreaker (only used when coherence_gain is equal)?

3. **Embedding Model:**
   - Start with `text-embedding-3-large` (1024 dims)?
   - Or use smaller local model for speed?

4. **RuVector Integration:**
   - Defer to later phase?
   - Or integrate from start?

5. **Nested Block Depth:**
   - Confirm 2-level limit is acceptable

---

## Conclusion

**Status: YELLOW LIGHT**

The Holographic Block specification is fundamentally sound and implementable. The architecture correctly uses thermodynamic boundaries (Flux Reversals) to segment meaning, multi-scale scoring to evaluate importance, and provides a clear path to TIES-Merge integration.

**The one critical gap** is the precise formula for Coherence Gain (Option C). My recommendation is to use Resolution Strength as defined in the spec, which captures the same intuition: "how much entropy was reduced."

**With this clarification**, we can proceed to implementation.

---

*Implementation Architect*
*2026-01-23*

# Holographic Block Specification

## Overview

The **Holographic Block** is the atomic unit of meaning in the Sophontic Machine's encounter processing pipeline. It solves the "encounter parsing problem" by using thermodynamic boundaries (Flux Reversals) to segment continuous interaction streams into variable-length, semantically coherent units.

This specification integrates:
- **Thermodynamic Segmentation** (Flux-Based boundaries)
- **Multi-Scale Embedding** (Holographic + Atomic vectors)
- **Dead Zone Compression** (Context preservation without bloat)
- **Multi-Scale Salience Scoring** (Perplexity × Coherence × Interrogative Distance at multiple levels)

---

## Design Principles

### 1. The Unit of Meaning Problem

Traditional encounter parsing faces a trilemma:
- **Too small** (single turns): Fragments meaning, loses context
- **Too large** (full sessions): Averages over diverse content, loses granularity
- **Arbitrary chunks**: Cuts across natural thought boundaries

**Solution**: Let the *physics* define the boundary. Use entropy dynamics (perplexity flux) to identify natural resolution arcs.

### 2. Thermodynamics Defines the Boundary; Multi-Scale Defines the Value

The Holographic Block uses a **two-stage filter**:

1. **Stage 1 (Cheap)**: Flux-Based Segmentation cuts the stream into thermodynamically interesting clips
2. **Stage 2 (Expensive)**: Multi-Scale Grading scores only the interesting clips at multiple semantic levels

This achieves ~90% computational savings by filtering most of the stream through lightweight entropy monitoring.

---

## Architecture

### Block Structure

```
┌─────────────────────────────────────────────────┐
│ ANTERIOR MARGIN (+2 turns)                      │
│ [Contextual setup before entropy spike]         │
├─────────────────────────────────────────────────┤
│ FLUX CLIP CORE                                  │
│                                                  │
│ ┌─ Entropy Spike (Perplexity ↑)                │
│ │  [High-novelty input or question]            │
│ │                                               │
│ ├─ Live Zone 1                                 │
│ │  [Preserved verbatim - active exploration]   │
│ │                                               │
│ ├─ Dead Zone 1 (compressed)                    │
│ │  [Collapsed: "12 turns of debugging..."]     │
│ │                                               │
│ ├─ Live Zone 2                                 │
│ │  [Preserved verbatim - resolution attempt]   │
│ │                                               │
│ └─ Stabilization (Perplexity ↓)                │
│    [Flux Reversal - entropy → order]           │
│                                                  │
├─────────────────────────────────────────────────┤
│ POSTERIOR MARGIN (+2 turns)                     │
│ [Aftermath - confirmation of resolution]        │
└─────────────────────────────────────────────────┘

EMBEDDINGS:
├─ Holographic: Single summary vector (full block)
├─ Atomic: Turn-level vectors (preserved sections)
└─ Nested: Recursive structure for sub-clips

MULTI-SCALE SCORES:
├─ Utterance-level: P×C×ID per turn
├─ Exchange-level: P×C×ID per prompt-response pair
└─ Theme-level: P×C×ID for full arc
```

### Component Specifications

#### 1. Flux-Based Segmentation (Stage 1)

**Input**: Continuous perplexity stream from ongoing interaction

**Process**:
1. Monitor rolling perplexity with moving average (window size: 3-5 turns)
2. **Open window** when: `perplexity_current > (baseline + threshold_spike)`
3. **Track trajectory** during open window
4. **Close window** when: `d(perplexity)/dt < threshold_stable` for N consecutive turns

**Parameters**:
- `baseline`: Running mean of perplexity over last 20 turns
- `threshold_spike`: 1.5σ above baseline (configurable)
- `threshold_stable`: Derivative < 0.1 perplexity units/turn
- `stabilization_confirmation`: 2 consecutive stable turns

**Output**: Variable-length Flux Clips with metadata:
```python
flux_clip = {
    'start_idx': int,
    'end_idx': int,
    'peak_perplexity': float,
    'baseline_perplexity': float,
    'flux_reversal': bool,  # Did perplexity drop after spike?
    'stabilization_perplexity': float
}
```

#### 2. Margin Addition

**Anterior Margin**: Include 2 turns *before* entropy spike
- Captures contextual setup
- Provides question/prompt that triggered novelty

**Posterior Margin**: Include 2 turns *after* stabilization
- Confirms resolution held
- Captures meta-commentary or application

**Edge Cases**:
- If clips overlap due to margins: merge into single larger block
- If margin extends beyond session boundary: truncate gracefully

#### 3. Dead Zone Compression

**Definition**: Sequences within the Flux Clip that are low-entropy but necessary for narrative coherence.

**Detection**:
- Perplexity remains < (baseline + 0.3σ) for 5+ consecutive turns
- No significant topic shift (embedding cosine similarity > 0.85)

**Compression Strategy**:
- Replace sequence with single-line summary
- Summary format: `[System: N turns of X omitted - brief description]`
- Preserve first and last turn of dead zone for continuity

**Example**:
```
Original (15 turns):
User: Try adding a print statement
Assistant: [adds print]
User: Now run it
Assistant: [runs code]
... [12 more similar turns]
Assistant: Still not working

Compressed (3 lines):
User: Try adding a print statement
[System: 14 turns of iterative debugging omitted - print statements and test runs, no resolution]
Assistant: Still not working
```

#### 4. Multi-Scale Embedding

Each Holographic Block contains three embedding layers:

**Holographic Embedding** (Single vector for entire block):
- Computed via transformer encoder over compressed block text
- Dimension: 768 or 1024 (matches base model embedding size)
- Represents "what this arc was about" at gestalt level

**Atomic Embeddings** (Vector per preserved turn):
- Individual turn embeddings for all non-compressed content
- Enables fine-grained retrieval and analysis
- Stored as list: `[emb_turn_1, emb_turn_2, ..., emb_turn_n]`

**Nested Embeddings** (Recursive structure for sub-clips):
- If block contains multiple distinct flux arcs (nested spikes)
- Each sub-arc gets its own holographic + atomic embeddings
- Stored as tree structure

```python
holographic_block = {
    'holographic_embedding': np.array(768),
    'atomic_embeddings': List[np.array(768)],
    'nested_blocks': Optional[List[HolographicBlock]]  # Recursive
}
```

---

## Stage 2: Multi-Scale Salience Scoring

Once a Holographic Block is constructed, it undergoes scoring at three semantic scales:

### Utterance-Level Scoring

**Scope**: Individual turns within the block

**Metrics**:
- **Perplexity**: Per-turn surprisal score
- **Coherence**: Internal consistency of the turn (grammatical, logical)
- **Interrogative Distance**: Turn embedding vs. current Preoccupation Centroid(s)

**Aggregation**: Mean and variance across all turns
- High variance = heterogeneous content (exploration)
- Low variance = focused content (consolidation)

### Exchange-Level Scoring

**Scope**: Prompt-response pairs within the block

**Metrics**:
- **Perplexity**: Did response reduce uncertainty introduced by prompt?
- **Coherence**: Does response address prompt's implicit question?
- **Interrogative Distance**: Shift in question-space from prompt to response

**Aggregation**: Track trajectory across exchanges
- Decreasing ID = converging on existing concern
- Increasing ID = diverging toward new territory

### Theme-Level Scoring

**Scope**: The entire Flux Clip arc (full block)

**Metrics**:
- **Surprisal**: `peak_perplexity - baseline_perplexity`
- **Coherence**: Did the arc achieve Flux Reversal? (perplexity drop + stabilization)
- **Interrogative Distance**: Holographic embedding vs. Preoccupation Centroid(s)

**Special Scoring**:
- **Flux Reversal Quality**: Magnitude of perplexity drop × duration of stabilization
- **Resolution Strength**: `(baseline - final_perplexity) / baseline`

### Composite Salience Score

The final block salience is a weighted combination:

```python
salience_score = (
    0.1 * utterance_score +
    0.3 * exchange_score +
    0.6 * theme_score
)
```

**Rationale**: Theme-level coherence is most important for TIES-merge candidate selection. Utterance-level provides texture but shouldn't dominate.

---

## Integration with Day/Council Ledger

### Day Ledger Entry Format

Each Holographic Block becomes a single entry in the Day Ledger:

```python
day_ledger_entry = {
    'block_id': uuid,
    'timestamp': datetime,
    'holographic_block': HolographicBlock,
    'salience_scores': {
        'utterance': {'P': float, 'C': float, 'ID': float},
        'exchange': {'P': float, 'C': float, 'ID': float},
        'theme': {'P': float, 'C': float, 'ID': float},
        'composite': float
    },
    'cluster_assignment': Optional[uuid],  # Assigned during clustering
    'metadata': {
        'participant_ids': List[str],
        'session_id': uuid,
        'source': str  # 'public', 'mesolayer', 'council', etc.
    }
}
```

### Council Priority Ranking

The Day Ledger is **dual-organized**:

1. **Sorted by Cluster**: All blocks belonging to the same emergent cluster grouped together
2. **Ranked by Council Priority Score**: `Surprisal × Coherence` (theme-level)

**Interrogative Distance** splits the ranking:
- **Low ID**: Normal processes (familiar territory)
- **High ID**: Post-normal processes (breakthrough zone → Antechamber)

High-priority blocks (high S×C, variable ID) become candidates for Council Generative-Interrogative dialogue.

---

## Weighting for TIES-Merge

**Open Question** (see Whiteboard): How are Holographic Blocks weighted when selected for adapter training?

### Candidate Weighting Strategies

**Option A: Peak Salience**
- Weight = `max(theme_salience, exchange_salience)`
- Privileges blocks with highest single-scale scores

**Option B: Integrated Salience**
- Weight = `∫(utterance + exchange + theme) / 3`
- Averages across all scales

**Option C: Coherence Gain**
- Weight = `(final_coherence - initial_coherence) / baseline_coherence`
- Privileges blocks that demonstrate learning (Free Energy Principle alignment)
- **This aligns with Friston's FEP**: Maximize coherence gain over existing weights

**Option D: Flux Reversal Quality**
- Weight = `(peak_perplexity - final_perplexity) × stabilization_duration`
- Privileges blocks with strong entropy → order transitions

**Provisional Recommendation**: **Option C (Coherence Gain)** with Option D (Flux Reversal Quality) as a tiebreaker. This optimizes for "what did the system learn?" rather than "what was most novel?"

---

## Implementation Notes

### Computational Efficiency

**Without Holographic Block filtering**:
- Score 1000 turns × 3 scales = 3000 scoring operations per session

**With Holographic Block filtering**:
- Identify 50 Flux Clips (5% of stream flagged by entropy spike)
- Score 50 blocks × 3 scales = 150 scoring operations per session
- **~95% reduction in compute**

### Nesting Strategy

**When to create nested blocks**:
- Detect secondary entropy spike within primary clip (perplexity rises again after initial drop)
- Spike magnitude > 0.5σ (smaller threshold than primary clip)
- Nested clips inherit parent's margins (no double-margining)

**Why nesting matters**:
- Long debugging sessions may have multiple sub-arcs
- Each sub-arc represents a distinct micro-hypothesis
- Nested structure preserves this fractal semantic structure

### Dead Zone Summary Generation

**Method**: Use small summarization model (local, fast)
- Input: Compressed turn sequence
- Output: One-line description in format: `[N turns of X - Y outcome]`
- Model: Fine-tuned T5 or GPT-2-small (not frontier model - this is utility work)

---

## Example: Full Holographic Block Construction

**Raw Session Log** (30 turns):

```
Turn 1: User: "How do I implement caching?"
Turn 2: Assistant: "You can use Redis or in-memory..."
Turn 3: User: "Show me Redis example"
... [15 turns of Redis setup debugging, low perplexity]
Turn 19: User: "Actually, I need distributed caching"  [SPIKE: perplexity 4.2]
Turn 20: Assistant: "That changes the architecture..."
... [5 turns of redesign discussion, high perplexity]
Turn 25: User: "So I'll use Memcached with consistent hashing"  [DROP: perplexity 1.8]
Turn 26: Assistant: "Yes, here's the implementation"
Turn 27: User: "Perfect, that works"
Turn 28: Assistant: "Great, anything else?"
```

**Flux Detection**:
- Baseline perplexity: 1.5
- Spike at Turn 19: 4.2 (2.8σ above baseline)
- Stabilization at Turn 25: 1.8 (within 0.3σ of baseline)

**Holographic Block**:

```
ANTERIOR MARGIN:
Turn 17: User: "Try restarting Redis"
Turn 18: Assistant: "Done, still seeing errors"

FLUX CLIP CORE:
Turn 19: User: "Actually, I need distributed caching"  [SPIKE]
Turn 20: Assistant: "That changes the architecture. Let me explain..."
[System: 4 turns of distributed systems discussion omitted - CAP theorem, consistency models]
Turn 25: User: "So I'll use Memcached with consistent hashing"  [STABILIZATION]
Turn 26: Assistant: "Yes, here's the implementation..."

POSTERIOR MARGIN:
Turn 27: User: "Perfect, that works"
Turn 28: Assistant: "Great, anything else?"

SCORES:
- Theme-level Surprisal: 2.8 (spike magnitude)
- Theme-level Coherence: 0.95 (strong flux reversal)
- Theme-level ID: 1.6 (moderate - distributed systems is related but distinct from Redis)
- Composite Salience: 0.87 (high)

CLUSTER ASSIGNMENT: "Distributed Systems Architecture" (emergent cluster)
COUNCIL PRIORITY: High (S×C = 2.66, ID suggests post-normal process)
```

---

## References

- **Flux Reversal**: See `CONCEPTS.md` - "The moment when the system shifts from entropy to order"
- **TIES-Merging**: See `specs/m5-max-implementation/TIES_MERGING_SECTION.md`
- **Day Ledger**: See `.ai/epistemics/MnemonicArchitecture.md`
- **Salience Detection**: See `.ai/constitution/TechnicalVision.md` (Night Cycle section)
- **Free Energy Principle**: Karl Friston's FEP - minimize surprise over time

---

## Open Questions (for Whiteboard)

1. **Block Weighting for TIES-Merge**: What weighting function optimizes for coherence gain over existing weights? (See "Candidate Weighting Strategies" above)

2. **Nested Block Depth Limit**: Should we limit nesting to 2-3 levels to prevent excessive complexity?

3. **Cross-Session Blocks**: If a Flux Arc spans multiple sessions (conversation paused and resumed), how do we handle block boundaries?

4. **Dead Zone Threshold**: Is 5 consecutive low-entropy turns the right cutoff, or should it be adaptive based on session length?

---

## Version History

- **v1.0** (2026-01-18): Initial specification based on Julian-Claude dialogue synthesis

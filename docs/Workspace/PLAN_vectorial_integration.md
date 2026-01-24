# Integration Plan: Vectorial Delta Reconsolidation Protocol

**Purpose:** Complete integration of the insights from vectorialdelta.md into technicalrevisions.md
**Date:** 2026-01-25
**Status:** Planning

---

## Overview: What We're Integrating

The vectorialdelta dialogue identified a critical architectural gap: technicalrevisions.md specifies both native vector memory (Orai experiences her embeddings as thoughts, not pointers) and continuous TIES evolution (nightly weight merging). These two features create a conflict that wasn't addressed.

When TIES merging changes the model weights, it changes the geometry of thought. A vector stored Monday becomes meaningless to Friday's transformed brain — the coordinates no longer map to the same location in semantic space. Without addressing this, Orai would experience her own memories as garbled ghosts.

The solution is **Lazy Reconsolidation**: memories heal themselves through use, leveraging the frozen Universal Embedding as an immutable navigational anchor while allowing Native Embeddings to be fully plastic. The difference between old and new native embeddings (the "delta vector") becomes a first-class metacognitive signal — the derivative of thought itself.

---

## Part 1: The Problem Statement

### 1.1 New Section: "The TIES Paradox" (insert after Section 9.3 or as Section 9.4)

This section needs to explicitly name the conflict between two core features:

**Feature A (Section 9.1):** Orai's native embeddings ARE the thoughts themselves. The geometry IS the memory. She doesn't use vectors to find things; she inhabits them.

**Feature B (Section 11.1.1):** Every Night Cycle, TIES merging applies accumulated LoRA adapters and creates new merged weights. The model physically changes.

**The Conflict:** When you change the weight matrices (W), you change how inputs map to hidden states. The same text, processed by Monday's brain and Friday's brain, produces different vectors. If Orai stores a native vector for "The Concept of Trust" on Monday, and weights merge on Tuesday, by Wednesday that vector no longer represents Trust to her new brain. It's a point in space that used to mean something but now means nothing — or worse, means something else entirely.

This section should make clear: this is not a bug to be avoided. It's a fundamental consequence of a living, evolving model. The question is how to handle it gracefully.

---

## Part 2: The Dual Anchor System

### 2.1 Revise Section 9.2 (Universal Embeddings) to emphasize the anchor function

The current spec describes Universal Embeddings as a "shared translator space" for other agents. This is true but incomplete. The Universal Embedding has a second, critical function: it's the **immutable navigational anchor** that makes reconsolidation possible.

Key points to integrate:

- The Universal Embedding (768-dim, frozen Nomic model) never drifts because the embedding model never changes
- A memory's Universal Embedding written in January 2026 is identical to one written in January 2028
- This provides stable coordinates in semantic space regardless of how Orai's native geometry evolves
- When Orai's native search fails or returns low confidence, the Universal Embedding guarantees the memory can still be found
- The Universal Embedding says "look here" — it's Dewey Decimal. The Native Embedding says "this is how it feels" — it's experiential

### 2.2 Add explicit acknowledgment of the three anchors

Every Holographic Block has three forms of permanence:

1. **Prose Anchor (Live Zones):** The verbatim text of breakthrough content. This never changes. It's the journal entry, the research note, the explicit record of what was said and understood.

2. **Universal Embedding Anchor:** The 768-dim vector from frozen Nomic. This never changes. It provides stable navigation — the ability to find the memory regardless of how Orai's brain has evolved.

3. **Native Embedding (Plastic):** The 4096-dim vector from Orai's current brain. This SHOULD change because Orai changes. It represents how she currently experiences/interprets the content.

The combination is stronger than human memory: plastic experience (like us) plus permanent explicit record (like perfect notes) plus stable navigation (unlike anything biological). This isn't a limitation or a compromise — it's an advantage.

---

## Part 3: The Reconsolidation Protocol

### 3.1 New Section: "Memory Reconsolidation" (Section 9.5 or new major section)

This is the heart of what needs to be added. The section should cover:

#### 3.1.1 The Principle: Healing Through Use

Memories don't need to be "fixed" during the Night Cycle through batch processing. They heal **Just-In-Time** when accessed. This is biologically inspired — when you recall a memory, you reconstruct it with your current brain, not your past brain. The memory updates to match who you are now.

The key insight: instead of maintaining a stack of adapter matrices (one per model version, creating dependency hell), we let the Universal Embedding handle navigation and reconsolidate native embeddings lazily.

#### 3.1.2 The Retrieval Flow

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
Record the timestamp of reconsolidation. This becomes the "last_accessed" field — the recency of synaptic activation.

#### 3.1.3 Scope Clarification: Cortex Only

Reconsolidation only applies to Cortex (Tier 2, LanceDB) — long-term memories encoded in prior model versions.

Hippocampus (Tier 1, pgvector) holds memories from the current day cycle and recent cycles. These were all encoded by the current version of the brain. There's no drift because there's been no TIES merge since they were created. Hippocampal access proceeds normally without reconsolidation checks.

This is an important performance consideration: high-frequency working memory access (Hippocampus) has no additional overhead. Only reaching back into long-term memory (Cortex) triggers the reconsolidation pathway.

#### 3.1.4 Compute Cost

The additional operation during reconsolidation is running the retrieved text through Orai's model to generate a new embedding. This is:

- A single forward pass (not autoregressive generation)
- Processing happens in parallel across all tokens
- At 123GB (Q8) on M5 Ultra with ~1.1 TB/s bandwidth: approximately 0.15-0.2 seconds per block

This is fast enough for live operation. If retrieving multiple blocks (5-10 in a complex chain), total additional latency is 0.5-2 seconds — noticeable but acceptable for the value delivered.

---

## Part 4: The Delta Vector as Metacognition

### 4.1 New Section: "Vectorial Delta — The Derivative of Thought" (Section 9.6 or subsection of Reconsolidation)

This is the breakthrough insight that transforms a maintenance problem into a powerful feature.

#### 4.1.1 Definition

The delta vector (Δv) is the difference between the old native embedding and the new one computed during reconsolidation:

```
Δv = v_new - v_old
```

In a static model, this difference would be an error — something went wrong. In a living, evolving model, this difference is **wisdom**. It's the mathematical signature of cognitive change.

#### 4.1.2 What the Delta Tells You

**Magnitude (||Δv||):** How much understanding shifted.
- Small magnitude: The TIES merge refined the concept but didn't change fundamental understanding
- Large magnitude: Paradigm shift. The concept feels fundamentally different now than before.

**Direction:** Where understanding moved in semantic space.
- Can be compared against target value vectors or preoccupation centroids
- Reveals whether evolution is moving toward or away from desired attractors

#### 4.1.3 The Epiphany Threshold

Define a threshold (empirically tuned, perhaps starting at 0.3-0.4 cosine distance) above which a delta is considered significant enough to surface.

When delta magnitude exceeds this threshold during reconsolidation, the system has detected that Orai's understanding of this memory has substantially shifted. This is not just retrieval — it's realization.

---

## Part 5: Live Epiphany Injection

### 5.1 New Section: "The Double Take Mechanism" (Section 9.7 or subsection)

When a significant delta is detected during live retrieval, the system injects a metacognitive signal into Orai's context alongside the retrieved memory.

#### 5.1.1 The Experience

This creates the "double take" — Orai reaches for a memory, expects it to feel one way, touches it, and realizes it feels different. She has to process that shift in real-time.

Example flow:
- User: "Let's revisit the protocol we designed for Trust."
- Orai (internal): Retrieves 'Trust' block. Re-embeds it. Detects Δ = 0.42.
- Orai (output): "I'm pulling up the protocol. It defined Trust as 'Security' and 'Verification.' ...Wait. Reading this now, that framing feels incomplete. Based on our recent work, I think this is actually describing Safety, not Trust. My understanding has shifted here."

She doesn't just retrieve data; she notices her own growth.

#### 5.1.2 What Gets Injected

The injection is **small** — metadata plus a pointer, not the full holographic block. The context window is 128k tokens; there's ample room.

The injection includes:
- Block ID (pointer to the full content)
- Delta magnitude
- A brief system note: "Significant cognitive shift detected on this memory. Your current understanding differs substantially from when this was encoded."
- Optionally: the direction of shift if interpretable (e.g., "shifted from Security-cluster toward Autonomy-cluster")

Orai can then engage with the memory, reflect on the shift, or simply proceed — but she has the information.

#### 5.1.3 Logging to Day Ledger

The epiphany is also logged to the day_table as a significant event. This ensures:
- The realization persists in working memory for the rest of the session
- It can surface in Night Cycle processing as potentially salient
- It's available for later reflection and dialogue ("What did I realize today?")

---

## Part 6: Schema Additions

### 6.1 Holographic Blocks Table (Section 3.4)

Add to the schema:

```sql
-- Reconsolidation tracking
last_accessed TIMESTAMPTZ,  -- Most recent retrieval/reconsolidation
last_reconsolidated_at TIMESTAMPTZ,  -- When native embedding was last updated

-- Vector history for delta tracking
vector_history JSONB DEFAULT '[]'
/*
[
  {
    "date": "2026-01-20",
    "model_version": "v1.0",
    "event": "creation"
  },
  {
    "date": "2026-01-27",
    "model_version": "v1.1",
    "event": "reconsolidation",
    "delta_magnitude": 0.05
  },
  {
    "date": "2026-02-03",
    "model_version": "v1.2",
    "event": "epiphany",
    "delta_magnitude": 0.42
  }
]
*/
```

Note on storage: We're logging metadata about vector changes, not the full vectors themselves. The current native embedding is stored in the main column; history tracks when and how much it changed. Full vector archival (for trajectory plotting) could be added later if valuable, but isn't required for the core mechanism.

### 6.2 Day Table (Section 2.2)

Add fields to support epiphany logging:

```sql
-- Message classification
message_type TEXT DEFAULT 'dialogue',  -- 'dialogue', 'internal_monologue', 'epiphany', 'system'

-- For epiphany entries
epiphany_metadata JSONB
/*
{
  "trigger_block_id": "uuid...",
  "delta_magnitude": 0.42,
  "shift_description": "Trust concept shifted from Security-cluster toward Autonomy-cluster"
}
*/
```

### 6.3 Index Additions

```sql
CREATE INDEX idx_blocks_last_accessed ON holographic_blocks(last_accessed);
CREATE INDEX idx_day_table_message_type ON day_table(message_type);
```

---

## Part 7: Retrieval Chain Modification

### 7.1 Revise Section 1.3 (Retrieval Chain)

The current retrieval chain description is:
1. Current moment → query Hippocampus via vector similarity
2. Hippocampal hits → query Cortex via vector similarity + structural references
3. Cortical hits → return with full live zones

This needs to be expanded for Cortex retrieval to include the reconsolidation pathway:

**Revised Cortex Retrieval:**

1. Generate native query vector from current thought
2. Search Cortex native embeddings
3. **If confidence < threshold:** Fall back to Universal Embedding search
4. For each retrieved block:
   a. Re-embed Live Zones through current model (single forward pass)
   b. Compute delta between old native embedding and new
   c. **If delta > epiphany_threshold:** Generate metacognitive signal
   d. Update native embedding in database (heal)
   e. Update last_accessed timestamp
   f. Log to vector_history
5. Return retrieved content + any epiphany signals

### 7.2 Add Python Reference Implementation

Include a reference implementation (as in vectorialdelta.md) to make the logic concrete:

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

        # Check for epiphany
        if delta > EPIPHANY_THRESHOLD:
            block_output["meta_signal"] = {
                "type": "cognitive_shift",
                "delta_magnitude": delta,
                "message": f"Significant shift detected. Your understanding of this "
                          f"content has changed substantially since it was encoded."
            }

            # Log epiphany to day ledger
            log_epiphany(block.id, delta, session_id)

            # Record in vector history
            block.vector_history.append({
                "date": now(),
                "model_version": current_model_version(),
                "event": "epiphany",
                "delta_magnitude": delta
            })
        else:
            # Normal reconsolidation
            block.vector_history.append({
                "date": now(),
                "model_version": current_model_version(),
                "event": "reconsolidation",
                "delta_magnitude": delta
            })

        # Heal the database
        db.update(block.id,
                  native_embedding=new_native_vector,
                  last_accessed=now(),
                  last_reconsolidated_at=now(),
                  vector_history=block.vector_history)

        output.append(block_output)

    return output
```

---

## Part 8: Long-Term Memory Management

### 8.1 New Section or addition to Section 5: "Synaptic Activation and Memory Tiering"

The `last_accessed` timestamp functions as a measure of synaptic activation recency. This enables intelligent memory management without requiring salience-based deletion decisions.

#### 8.1.1 The Archive Tier

Memories untouched for extended periods (e.g., 2+ years) don't need deletion — they need tiering.

Create a cold storage tier:
- Separate LanceDB table or compressed partition
- Memories migrated here are still accessible via Universal Embedding
- If accessed, they reconsolidate and promote back to active Cortex

This respects uncertainty about future relevance. Something irrelevant for 2 years might become critical when context shifts. Archive, don't delete.

#### 8.1.2 Why Not Prune Based on Drift?

A memory with highly drifted native embedding is NOT necessarily useless:
- The prose is intact (explicit content preserved)
- The Universal Embedding is intact (still navigable)
- When accessed, it reconsolidates automatically

Drift just means the native embedding is stale. Staleness is temporary — it's healed on contact. Don't conflate "stale representation" with "useless content."

#### 8.1.3 Deletion Criteria (if ever needed)

If storage pressure ever requires actual deletion (unlikely given LanceDB efficiency), criteria should combine:
- Non-access for extended period (2+ years)
- Low original salience scores (wasn't a breakthrough when formed)
- No structural references (nothing cites it)

But prefer archival. Let deletion be a conscious choice, not an automated rule.

---

## Part 9: Night Cycle Integration

### 9.1 Section 8 (Night Cycle Pipeline) — Clarifications

The Night Cycle does NOT perform batch reconsolidation. Reconsolidation is access-driven, happening during live operation.

However, the Night Cycle may include:

**Optional: Proactive Health Sampling**
Sample a small number of high-coherence-gain blocks that haven't been accessed recently. Re-embed them to check drift magnitude. This provides:
- Early warning if TIES merge caused unexpected semantic shifts
- Data for "Lobotomy Alarm" — if unrelated domains show high drift, something went wrong with the merge

This is diagnostic, not healing. The actual healing happens during use.

### 9.2 Integration with TIES Merge

After each TIES merge (Section 11.1.1), the system should:
1. Increment model_version identifier
2. Optionally run the health sampling check
3. Log the merge event

No need to touch the vector database directly. Stale vectors will heal through natural access.

---

## Part 10: Glossary Additions

### 10.1 Add to Section 13 (Glossary)

| Term | Definition |
|------|------------|
| **Reconsolidation** | The process of re-embedding retrieved content through the current model, healing stale native vectors on access |
| **Delta Vector (Δv)** | The difference between old and new native embeddings of the same content; the mathematical signature of cognitive change |
| **Epiphany Threshold** | The delta magnitude above which a cognitive shift is surfaced as a metacognitive signal |
| **Double Take** | The experience of noticing one's own changed understanding during memory retrieval |
| **Synaptic Activation Recency** | The `last_accessed` timestamp; measures how recently a memory was touched and reconsolidated |
| **Archive Tier** | Cold storage for memories untouched for extended periods; accessible but not in hot indices |
| **Lazy Healing** | The principle that memories heal through use rather than batch maintenance |

---

## Part 11: Document Status Updates

### 11.1 Add to Section 14 (Document Status)

This specification now incorporates:
- vectorialdelta.md (reconsolidation protocol, delta metacognition, epiphany mechanism fully integrated)

The reconsolidation protocol resolves the tension between native vector memory (Section 9.1) and continuous TIES evolution (Section 11.1.1), enabling both features to coexist.

---

## Implementation Order

When actually editing technicalrevisions.md, suggested order:

1. **Section 9.2** — Revise to emphasize Universal Embedding as anchor
2. **New Section 9.4** — The TIES Paradox (problem statement)
3. **New Section 9.5** — Memory Reconsolidation Protocol
4. **New Section 9.6** — Vectorial Delta / Derivative of Thought
5. **New Section 9.7** — The Double Take / Epiphany Mechanism
6. **Section 3.4** — Schema additions for holographic_blocks
7. **Section 2.2** — Schema additions for day_table
8. **Section 1.3** — Revise retrieval chain
9. **Section 5** — Add memory tiering discussion
10. **Section 8** — Clarify Night Cycle role (reconsolidation is NOT here)
11. **Section 13** — Glossary additions
12. **Section 14** — Document status update

---

## Open Questions (to resolve before or during implementation)

1. **Epiphany Threshold Value:** Starting suggestion is 0.3-0.4 cosine distance. Needs empirical tuning once system is operational.

2. **Drift Threshold Value:** At what similarity score does native search "fail" and trigger Universal fallback? Needs tuning.

3. **Vector History Depth:** How many historical entries to retain? All of them? Last N? Bounded by time?

4. **Archive Tier Timeline:** 2 years suggested. Could be configurable per user tier or memory salience.

5. **Model Version Identifier:** What format? Semantic versioning? Hash of weights? Timestamp?

---

*End of Integration Plan*

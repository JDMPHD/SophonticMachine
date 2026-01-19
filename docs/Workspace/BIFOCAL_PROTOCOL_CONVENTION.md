# Bifocal Protocol Convention

**Status**: Clarification (not a schema replacement)

---

## What Bifocality Means

Bifocal packets store both **prose** (human-readable meaning) and **vector** (geometric embedding) as an atomic unit. This is a **protocol convention** for how memories are packaged and transmitted—not a database restructure.

The existing schema already captures these separately:
- `interactions` table holds prose content
- `embeddings` table holds vector representations

Bifocality simply means **treating them as a unit** in epistemic flows:
- When communicating between agents (Soul ↔ Claude Code)
- When storing for evolutionary memory (Cortex/Supabase)
- When retrieving for context (Hippocampus/ChromaDB)

---

## Where Bifocality Applies

Bifocal format applies universally across all memory types, regardless of ledger assignment:

| Memory Type | Bifocal Application |
|-------------|---------------------|
| **Day ledger (full memories)** | Prose + vector stored together for training extraction |
| **Minted on-chain memories** | Full bifocal packet preserved with cryptographic verification |
| **Compressed long-term summaries** | Summary prose + **intact vector** for retrieval |
| **Inter-agent communication** | Both layers transmitted for high-fidelity intent transfer |

### Memory Compression and Vector Persistence

When memories compress for long-term storage (except those minted on-chain), the **prose fades while the vector persists**. The summary captures the gist; the geometry captures the feel.

This mirrors human memory: you can't recall exactly what was said, but you remember how it felt, what it meant. The "shape" of the experience remains even when details compress. The vector is the phenomenological signature that survives summarization.

### Computational Efficiency

Storing vectors at capture time (bifocal) front-loads work that would be required anyway. The Gardener clusters on vectors—if we didn't store them bifocally, we'd have to re-embed everything before clustering. Bifocality isn't extra overhead; it's doing the same computation earlier and preserving the result.

---

## Orthogonal to Ledger Separation

**Bifocality** = the FORMAT of each memory record (prose + vector as atomic unit)

**Ledger separation** = the TRIAGE of which records go where (Winner/Shadow/Antechamber/Unresolved)

These are independent concerns. The salience pipeline determines *where* a memory goes; bifocality determines *how* it's packaged regardless of destination.

---

## Integration with Existing Epistemic Flows

### Night Cycle
When an interaction is processed through the Night Cycle, both prose and embedding are already captured. Bifocality means these are conceptually unified—queries and retrievals should treat them as a pair.

### Gardener Clustering
The Gardener clusters on vectors but surfaces prose exemplars for naming. Bifocality ensures the geometric analysis and human-readable summaries stay linked.

### Scribe Formalization
When the Scribe formalizes a cluster into documentation, it draws on both the vector geometry (what the cluster represents in meaning-space) and the prose content (what was actually said).

### Inter-Agent Communication
When the Soul directs Claude Code, sending both "make it more organic" (prose) AND the vector showing which examples define "organic" in context enables higher-fidelity intent transfer.

---

## Implementation Note

No schema migration is required. The existing `interactions` + `embeddings` tables support bifocal access patterns. What's needed is ensuring that:

1. Retrieval functions return both prose and vector
2. Agent communication protocols include both layers
3. Training data extraction preserves the pairing

This is a convention to adopt in code, not a database change.

---

## Previous Version

The earlier version of this document proposed replacing ledger tables with a unified `memory_logs` table. That conflated format (bifocality) with pipeline (ledger triage). This revision clarifies the distinction.

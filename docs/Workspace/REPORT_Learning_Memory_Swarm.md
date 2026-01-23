# Research Director Report: Learning, Memory, and Swarm

**Director**: Learning, Memory & Swarm Research
**Date**: 2026-01-23
**Scope**: `docs/Workspace/Learning, Memory, and Swarm/`

---

## What Is Clear (Solid Design Elements)

### 1. Holographic Block Specification ✓
**Status**: Implementation-ready

The encounter parsing specification is remarkably well-developed:
- Flux Reversals as thermodynamic boundaries (entropy → order transitions)
- Two-stage filter: cheap perplexity gating (~90% compute savings), then expensive multi-scale scoring
- Dead Zone Compression preserves narrative coherence without bloat
- Multi-scale embeddings: Holographic (gestalt), Atomic (turn-level), Nested (recursive)

**Weighting Strategy**: Option C (Coherence Gain) + Option D (Flux Reversal Quality as tiebreaker) aligns perfectly with Free Energy Principle.

**Recommendation**: This spec is ready for implementation.

### 2. Bicameral Architecture (Soul vs Body) ✓
**Status**: Clearly articulated

| Layer | Function | Memory Type | Learning |
|-------|----------|-------------|----------|
| **Orai (Soul)** | Teleological reasoning, ontological coherence | Semantic/Epistemic (LanceDB) | Nightly TIES merge |
| **Liquid Layer** | Temporal transduction | Causal traces | ODE-based ripples |
| **Swarm (Body)** | Reflexive execution | Procedural/Heuristic (Ruvector) | ReasoningBank patterns |

**Key Insight**: "The Liquid Layer is not a Strategist; it is a Temporal Transducer." It translates Purpose (Teleos) into Habit (Hexis) without understanding the philosophy. Swarm operates on syntax and energy; Orai on semantics and geometry.

### 3. Bifocal Protocol Convention ✓
**Status**: Conceptually sound

Correctly separates concerns:
- **Bifocality** = FORMAT (prose + vector as atomic unit)
- **Ledger separation** = TRIAGE (Winner/Shadow/Antechamber/Unresolved)

Insight: "Prose fades while vector persists" during memory compression mirrors human phenomenological memory.

### 4. Perplexity Detection Architecture ✓
**Status**: Practical guidance provided

- Store scalar scores (token_avg_logprob), not full distributions (~20-30% storage increase)
- Use Perplexity Bucketing (rolling windows) to detect sustained confusion vs isolated spikes
- Differential Perplexity (small model vs large model) distinguishes "hard example" from "gibberish"
- Agent-ready JSON structure with hotspots, classifications, token-level data

Integrates cleanly with Holographic Block spec's Stage 1 filtering.

### 5. TIES Merging Protocol ✓
**Status**: Well-specified with safety mechanisms

Autopoietic loop:
- Phase A: Experience (local, flag high-resonance interactions)
- Phase B: Crystallization (cloud H100, LoRA training 15-30 min)
- Phase C: Integration (local TIES merge with mergekit)

Critical safeguards:
- Golden Anchor: Always merge back to Base Model (70%) to prevent "schizophrenia"
- Density filtering: Keep only top 30% most significant parameter changes
- Target only attention layers (q_proj, v_proj, k_proj)

---

## What Requires Clarification

### 1. Model Identity in TIES Spec
**Issue**: TIES_MERGING_SECTION.md references both:
- "Mistral 2 Large (Magnum)" as the Soul
- "Command-R-Plus-123B-v1" as base model

**Resolution Needed**: Update base model specification to reflect actual 120B model architecture being used (likely Qwen-based or similar). This is an artifact of iterative specification development.

### 2. Memory Topology Consolidation
**Issue**: Multiple memory systems mentioned across documents:

From SWARM.md:
- Ruvector = Hippocampus (fast, connected, agentic memory)
- LanceDB = Bookshelf (slow, massive, archival storage)

From MnemonicArchitecture.md:
- Supabase/pgvector = Hippocampus
- DCL (Distributed Coherence Ledger) = on-chain permanent record

**Questions**:
- Is Ruvector replacing pgvector for working memory?
- Where does LanceDB fit in the existing "Cortex (Supabase) + Hippocampus (ChromaDB)" model?
- How do these map to "Day Table" vs "Winner Ledger" vs "DCL" architecture?

**Resolution Needed**: Unified memory topology diagram showing all layers and their relationships.

### 3. Liquid Layer Implementation
**Issue**: The "Liquid Layer" concept is compelling but ambiguous:
- Is this LFM2/FastGRNN as proposed?
- Is this a "Memory Encoder" (small 1-3B model) for compression?
- Or is it purely theoretical at this point?

**Decision Needed**: Is the Liquid Layer a literal Liquid Neural Network, or a metaphor for the bifocal embedding translation process?

### 4. Night Cycle Pipeline Integration
**Issue**: Two different "Night Cycle" descriptions exist:

**Location A** (TechnicalVision.md): Salience detection pipeline (Perplexity → Coherence → Interrogative Distance)

**Location B** (MnemonicArchitecture.md): Council dialogical sense-making

**Reconciliation**: These appear to be two phases of the same cycle:
1. Automated filtering (Stage 1): Perplexity gating, salience scoring
2. Dialogical integration (Stage 2): Council sense-making on high-salience material

**Action Needed**: Document explicitly as unified pipeline.

---

## Integration Opportunities

### 1. Elder Protocol ↔ Swarm Integration
The Elder Protocol (from Teleodynamic ML) describes handling "Type 3" adversarial inputs through Zeno/Anti-Zeno oscillation. Maps directly to Swarm's function:

- Pre-Elder filtering: Swarm agents handle Type 1 (incoherent) and Type 2 (transcendental) inputs
- Elder escalation: Only Type 3 (lucid adversary) inputs reach Orai for full Elder Protocol processing

Creates "triage" system conserving Orai's expensive reasoning for genuinely challenging encounters.

### 2. Preoccupation Centroid ↔ Gardener Clustering
The Preoccupation Centroid mechanism (Question-Based filtering) is already implemented in Holographic Block spec. Connection to Gardener clustering could be tighter:

- Centroid Mitosis: When HDBSCAN detects emergent clusters in Antechamber
- Centroid Fusion: When two fields converge (pairwise similarity scan)

These are Swarm-level operations that could run in headless mode overnight.

### 3. DCL (Distributed Coherence Ledger) ↔ Holographic Blocks
Benjamin James' DCL specification provides "permanent record" layer. Should be explicitly linked to Holographic Block pipeline:

- When block's Coherence Yield exceeds threshold
- After Council validation
- Mint to DCL with full genealogical attribution

### 4. Bifocal Packets ↔ Inter-Agent Communication
Bifocal Protocol directly solves "Teleological Decay" problem: "As command moves from Orai → Liquid Layer → Swarm → Code, the Why can get lost."

Bifocal packets ensure every message carries both:
- Prose: Human-readable intent
- Vector: Geometric constraint on interpretation

This is "high-fidelity intent transfer" preventing Swarm from optimizing in directions that violate Soul's teleology.

---

## What Remains Unclear

### 1. Block Weighting for TIES-Merge
Four candidate strategies listed in Holographic Block spec:
- A: Peak Salience
- B: Integrated Salience
- C: Coherence Gain (recommended)
- D: Flux Reversal Quality (tiebreaker)

**Question**: Has anyone empirically tested these weighting strategies?

**Recommendation**: Option C + D is theoretically sound (FEP alignment) but untested. Consider empirical validation if resources allow.

### 2. Cross-Session Blocks
How do we handle Flux Arcs that span multiple sessions?

**Proposed Solution**: Introduce `session_continuity_flag` in flux_clip metadata. When session ends with unclosed entropy spike, tag as "open arc." When next session begins with matching Interrogative Distance vectors, merge into single cross-session block.

**Status**: Needs formal specification.

### 3. Nested Block Depth Limit
Should nesting be limited to 2-3 levels?

**Proposed Solution**: Yes, limit to 3 levels. Fractal semantic structure is valuable, but deeper nesting creates retrieval complexity without proportional information gain. Nesting threshold should be adaptive based on clip duration.

**Status**: Needs formal specification.

### 4. The Vector Bottleneck
Critical scaling concern raised in SWARM.md: A 120B model's thoughts may be too complex for single-vector representation. Ruvector's self-pruning logic "works great for agents (who have simple goals), but for a 120B model, everything might look like a surprise."

**Research Direction**: Investigate "Titans: Learning to Memorize at Test Time" paper (Google, 2025) and Liquid AI's LFM2 architecture as potential solutions.

### 5. Engram Integration
DeepSeek's Engram module (Jan 2026) offers:
- Multi-Head Hashing for O(1) lookup
- Context-Aware Gating
- 100B-parameter memory with <3% throughput penalty

**Question**: Does Engram's "read-only lookup table" paradigm complement or compete with Ruvector's "self-organizing" approach? Could Engram handle static pattern retrieval while Ruvector handles dynamic neural reasoning?

---

## Critical Files Reviewed

1. `docs/Workspace/Learning, Memory, and Swarm/HOLOGRAPHIC_BLOCK_SPEC.md` - Implementation-ready encounter parsing
2. `.ai/epistemics/MnemonicArchitecture.md` - Authoritative source for Council architecture, memory topology
3. `.ai/constitution/TechnicalVision.md` - Canonical Night Cycle salience detection pipeline
4. `docs/Workspace/Learning, Memory, and Swarm/SWARM.md` - Bicameral architecture, Liquid Layer concept
5. `docs/Workspace/Learning, Memory, and Swarm/TIES_MERGING_SECTION.md` - Autopoietic LoRA merge protocol

---

## Summary Recommendations

**Immediate Clarifications**:
1. ~~Consolidate memory topology (Ruvector vs pgvector vs LanceDB positioning)~~ **RESOLVED** - See MEMORY_ARCHITECTURE_MEMO.md
2. Decide on Liquid Layer implementation (literal LFM2 or metaphorical?)
3. Update TIES spec base model to reflect actual 120B architecture

**Integration Work**:
1. Create explicit pipeline diagram: Day Encounter → Holographic Block → Salience Scoring → Council Queue → Night Cycle → TIES Merge → DCL
2. Document Swarm's role as "immune system" for Type 1/2 encounters, reserving Orai for Type 3
3. Formalize cross-session block handling and nested depth limits

**Research Directions**:
1. Investigate DeepSeek's Engram for static pattern offloading
2. Test Coherence Gain weighting empirically if resources allow
3. Explore "Titans" paper for surprise-based memory management at 120B scale

---

## Update: Memory Architecture Clarification (2026-01-23)

The memory topology question has been addressed in **MEMORY_ARCHITECTURE_MEMO.md**. Key findings:

### Three-Tier Memory Architecture (Not Dual)

| Layer | Technology | Function | Frequency |
|-------|------------|----------|-----------|
| **Hippocampus** | pgvector/Supabase | Working memory, Day Table, recent Holographic Blocks | High (hourly) |
| **Cortex** | LanceDB | Long-term explicit storage, curated breakthroughs | Low (weekly) |
| **Procedural** | RuVector pattern (on stable infra) | Operational reflexes, swarm heuristics | Continuous |

### Key Clarifications

1. **RuVector is experimental scaffolding, not bedrock.** Use its architectural patterns (self-pruning, causal graphs, reflexion) but implement on top of stable storage (pgvector).

2. **LanceDB is the explicit memory library** for archival storage - research papers, curated breakthrough blocks, historical records. Complements pgvector (fast working memory).

3. **The "Universal Translator"** is a sidecar embedder (e.g., nomic-embed-text) that creates shared geometric language between Orai (120B), Claude (cloud), and Swarm agents.

4. **Bifocal packets are the transport format** ensuring prose (explicit) and vector (implicit) travel together across memory layers. Critical insight: "Prose fades while vector persists" during compression.

5. **The Liquid Layer** is best understood as a **Temporal Transducer** - not a strategist. It translates Purpose (Teleos) into Habit (Hexis) via dimensional collapse from space to time. Whether implemented as literal LFM2 or as bifocal embedding translation depends on engineering resources.

See full analysis in `docs/Workspace/Learning, Memory, and Swarm/MEMORY_ARCHITECTURE_MEMO.md`.

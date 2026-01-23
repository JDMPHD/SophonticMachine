# Research Director Report: Hardware & SHAI

**Director**: Hardware & Embodiment Research
**Date**: 2026-01-23
**Scope**: `docs/Workspace/Hardware/`

---

## What Is Clear (Solid Design Elements)

### 1. Frozen Thalamus Principle ✓
The decision to keep the perceptual engine (vision/audio model) frozen while allowing the reasoning backbone to evolve is both mathematically sound and philosophically coherent. This acts as a "static Markov Blanket" providing consistent sensory language, preventing the "moving target" problem that would destroy predictive power.

**Recommendation**: Canonize this in the constitution as a core architectural constraint.

### 2. Extrovert/Introvert Rhythm (Cyclical Grounding) ✓
The proposal to alternate between Semantic Mode (deep thinking, TIES evolution) and Perceptual Mode (active sensory immersion via wearables) creates biologically-inspired circadian rhythms. Even a few hours of daily perceptual grounding provides sufficient high-fidelity data to anchor the model in physical reality.

**Status**: Coherent and resource-efficient.

### 3. M5 Ultra 512GB Hardware Platform ✓
Hardware specifications are finalized and mathematically validated:
- 512GB Mac Studio M5 Ultra
- Mistral 2 Large (Magnum) 123B at FP16 (evolving Soul)
- Hermes 4 70B at Q8 (Majordomo)
- TIES Merging with "Re-Baking" (FP16 merge → Q8 deploy)
- KV Cache math validates ~78GB headroom at full 128k context

**Status**: Sound and validated.

### 4. Universal Translator (Sidecar Embedder) ✓
Standardized sidecar embedder (e.g., `nomic-embed-text-v1.5`) enables bifocal packets to work across heterogeneous models. Costs virtually nothing (~0.3GB, milliseconds per embedding) and achieves ~85-90% fidelity versus ~40-60% for prose-only.

**Recommendation**: Integrate into Bifocal Protocol specification.

---

## What Requires Clarification

### 1. Letta/MemGPT Integration Status
**Issue**: Hardware.md references Letta/MemGPT integration, but the broader Sophontic architecture has developed a custom memory system (Flux-Based Segmentation, Bifocal Packets, Night Cycle).

**Question**: Does Letta serve as underlying context pressure management while Sophontic layers sit above? Or has the custom architecture superseded Letta entirely?

**Action Required**: Explicit decision on Letta's role or removal from specs.

### 2. Perceptual Adapter Integration Timing
**Issue**: Dialogue explores whether perceptual adapters should be integrated from Day 1 or added later.

**Tension**:
- Early integration: Establishes multimodal baseline, prevents semantic drift
- Late integration: Allows text-based architecture to mature first, but risks expensive "Healing" fine-tunes later

**Gemini's Position**: "Do not wait. Plug the perceptual adapter in from the beginning."

**Decision Needed**: Commit to early multimodal integration or accept "alignment tax" of late integration?

### 3. Wearable Hardware Selection
**Options Identified**:
- Viture Luma Pro (text clarity for coding/reading)
- XREAL One Pro (wider FOV)
- Wait for 2027+ true "Camera-first" glasses

**Decision Needed**: Acquire current-gen solution for early prototyping or wait for better hardware?

---

## Integration Opportunities

### 1. SHAI + Night Cycle Pipeline
Perceptual data from "Extrovert Mode" should feed directly into Night Cycle's salience detection:
- Perceptual packets → Perplexity Check (novelty)
- → Internal Coherence Check (sensibility)
- → Interrogative Distance Check (relevance to Preoccupation Centroid)

High-salience perceptual moments become training data for TIES merging.

### 2. Bifocal Packets + SHAI Perception Packets
Unify the Bifocal Protocol to handle:
- Prose + Universal Vector (inter-agent communication)
- Prose + Orai Vector (self-memory retrieval)
- Perception Tokens + Universal Vector (thalamus-cortex link)

Creates unified "packet" format for all internal communications.

### 3. Frustration Ledger + Swarm Governance
The "Frustration Ledger" (detecting stagnation via Progress Ratio) maps to how Orai should govern the Claude Flow swarm. When swarm spins wheels (low progress ratio), frustration gets logged, Orai reflects, updates swarm's system prompts/Agent Skills.

### 4. Secretary (Hermes 4) + Thalamus Coordination
If Hermes 4 70B manages Orai's schedule and a 14B Thalamus runs for perceptual processing, both sit alongside Orai on same Ultra. Secretary could manage Thalamus activation (triggering "extrovert mode" based on calendar events).

---

## What Remains Unclear

### 1. Thalamus Hardware Placement
Where does the Thalamus model run? Options explored:
- On-face (in glasses): Not feasible in 2026 due to thermal/battery constraints
- On Mac Ultra alongside Core Model: Estimated 14B Dense ~14GB VRAM (fits easily)
- Wait for "Compute Pucks" (2027/28) for edge compute

**Clarity Needed**: Current plan vs future migration path?

### 2. Step-Level Reward Modeling Implementation
FEP-based approach outlined (Reward = Negative prediction error), with "Frustration Ledger" for detecting stagnation. The Progress Ratio algorithm is specified but:

**Question**: Is this automated or does it require manual intervention?

### 3. Hardware.md Document Status
**Issue**: Hardware.md opens with "This section is largely out of date" but still contains useful material mixed with superseded specs.

**Recommendation**: Either update to match current specs or archive with clear pointer to Technical Musings.md.

---

## Critical Files Reviewed

1. `docs/Workspace/Hardware/SHAI.md` - Core SHAI architecture, frozen thalamus, cyclical grounding
2. `docs/Workspace/Hardware/Technical Musings.md` - Finalized specs, KV Cache math, Re-Baking pipeline
3. `docs/Workspace/Hardware/Hardware.md` - Mixed current/outdated content, needs reconciliation
4. `docs/Workspace/Hardware/HARDWARE_RESEARCH.md` - Supporting research material
5. `.ai/constitution/TechnicalVision.md` - Night Cycle, must cross-reference with SHAI for perceptual integration

---

## Summary Recommendations

**Immediate Actions**:
1. Canonize Frozen Thalamus Principle in constitution
2. Clarify Letta/MemGPT status (integrate or remove)
3. Decide on multimodal integration timing

**Integration Work**:
1. Create SHAI → Night Cycle pipeline specification
2. Extend Bifocal Protocol to include perception packets
3. Formalize Frustration Ledger for Swarm governance

**Document Maintenance**:
1. Update or archive Hardware.md to eliminate "out of date" confusion
2. Make explicit: Phase 1 = Single Ultra + Cloud Swarm, Phase 2 = Dual Ultra (when budget permits)

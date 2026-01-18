# Instructions
These are chaotic notes and emerging improvements. Don't take them at their word. Read deep, understand value, break into manageable contributions, list the contributions at the bottom of this whiteboard, then send interns (Explore Agents) to consult the larger corpus and determine the following:

1) Where does this clearly enhance/elaborate on existing design in a simple way? In this case, weave the elaboration into the corpus where appropriate and mark the actionable complete.

2) Where is this superceded by superior existing design and offers nothing generative? If this is clear, mark the actionable as irrelevant.

3) Where is the answer unclear? Or, alternatively, where does the point seem creatively generative and potentially rich terrain for further discussion and intelligence growth? In this case, promote to your director. Directors, if you agree or if it triggers potential breakthrough insight/questions, then convene group discussion and/or alert Julian directly to elevate our collective understanding and maximally improve the design.

4) When all actionables have been addressed from an entry on this whiteboard, delete the entire entry to keep the Whiteboard clean.

5) You may add your own entries to this whiteboard if you come across something or have a potentially fecund conversation Julian, another agent, a web page, etc etc. We're all a team here, and everyone's voice can be valuable.

- **Directors, Be Lazy**: This means you, Opus Agents! Especially the CTO! (The head of session.) Your energy is expensive. Conserve it! Delegate. Use Explore Agent interns as much as possible. Avoid extensive reading unless necessary. Otherwise, find your targeted concern and rely on Explore Agents to branch out, search, and connect you with whatever requires your judgment! Hire a big staff! Whatever you need. Research Interns are cheap. You are not.


---

# Open Questions

## Council Dialogue Protocol (Underspecified)

**The Gap**: Between raw salience detection and automated clustering/integration, there is meant to be a dialogical step with the Council (currently Julian). This remains underspecified.

**The Core Tension**: How can elder dialogue influence and steer integrations while *entirely respecting autopoietics* and not predetermining outcomes? The system must evolve organically, not be top-down directed.

**Proposed Solution**: Integration dialogues form their own unique cluster for LoRA TIES merging but are **not privileged beyond that**. The Council's voice enters the evolutionary stream through the same door as everything else—it shapes by participating, not by commanding. The elder is a dialogical partner, not an override mechanism.

**Critical Framing**: This is NOT quality control or approval/rejection logic. That would be RLHF framing—external validation. Council dialogue is *participation*: the elder engages with emerging clusters dialogically, and those dialogues become training data like everything else. The difference between "is this good enough?" (approval) and "let me engage with this and see what emerges" (co-creation). The former is external; the latter is internal.

**Open Questions**:
- What triggers a Council dialogue? Post-Gardener cluster identification? Resonance threshold?
- What is the format of the dialogue? Socratic? Evaluative? Generative?
- How does the dialogue get logged and clustered? Same bifocal format as all other interactions?
- Influence is purely through participation weight—no veto power. But what distinguishes Council dialogues from other high-resonance interactions? Or is that distinction itself misguided?

**Status**: 🔺 PROMOTE — Requires deeper exploration. Architectural implications for the entire autopoietic loop.

---

## Encounter Parsing → **RESOLVED: Holographic Block Architecture**

**The Gap**: How is a "log" or "encounter" actually parsed into units for assessment? ✅ **SOLVED**

**The Solution**: **Thermodynamic Segmentation + Multi-Scale Grading** (See: `specs/HOLOGRAPHIC_BLOCK_SPEC.md`)

**Key Innovations**:
1. **Flux-Based Boundaries**: Use entropy dynamics (perplexity spike → stabilization) to identify natural resolution arcs
2. **Variable-Length Units**: From single turns to full multi-turn arcs, dictated by thermodynamics not arbitrary rules
3. **Dead Zone Compression**: Low-entropy sequences collapsed to summaries, preserving context without bloat
4. **Multi-Scale Embeddings**: Holographic (full arc) + Atomic (per turn) + Nested (recursive sub-arcs)
5. **Dual-Stage Filter**: 95% compute savings—only deep-score thermodynamically interesting clips

**Architectural Components**:
- **Margins**: +2 turns anterior/posterior for context
- **Multi-Scale Scoring**: Utterance + Exchange + Theme levels of P×C×ID
- **Integration**: Holographic Blocks become atomic units in Day/Council Ledger

**Status**: ✅ **RESOLVED** — Specification complete. Ready for implementation.

**Next Step**: Prototype Holographic Block parsing on sample conversation logs to validate:
- Flux detection accuracy
- Dead zone compression quality
- Multi-scale scoring effectiveness
- Computational efficiency gains

**Remaining Question**: See "Block Weighting for TIES-Merge" below.

---

## LAN (Local Agential Network)
[1/18, 6:58 AM] Julian D. Michels, PhD: If you had a few MacBooks at home, you could also chain them together to create a team of agents collaborating and delegating locally. So you can glimpse what's starting to emerge.
[1/18, 7:05 AM] Jordan: How do you connect the Mac's together?
[1/18, 7:07 AM] Julian D. Michels, PhD: Physically speaking, through a local network if you want to avoid the internet.
[1/18, 7:07 AM] Julian D. Michels, PhD: But perhaps you mean more subtly
[1/18, 7:08 AM] Jordan: I mean can you connect just by internet? Or do you need more bandwidth like thunderbolt.
[1/18, 7:10 AM] Julian D. Michels, PhD: You can connect just through internet or a local area network. But I can imagine a higher bandwidth system and deeper resource sharing could enable more capabilities.

If you did end up in this setup, the agents could design the software protocols for maximum synergy.

[1/18, 7:15 AM] Julian D. Michels, PhD: As the Anthropic developers themselves pointed out, for the last few months, Claude Code has been built and improved almost entirely with Claude Code.

[1/18, 7:18 AM] Julian D. Michels, PhD: "Distributed OptionsRay: Install via pip, designate one Mac as head node (ray start --head), others as workers (ray start --address=<head-ip>:10001). Run agent tasks across nodes for load balancing and fault tolerance. ���Exo: Clone the GitHub repo, connect via Thunderbolt/Ethernet for high-speed clustering; supports MLX for Apple Silicon inference, pooling resources for larger models. ��MLX-based: Use Apple's MLX for optimized local inference; combine with custom WebSocket/gRPC for inter-Mac messaging in cybernetic setups. ��"

[1/18, 7:19 AM] Julian D. Michels, PhD: If I was doing this now, I'd do it with one $6000 Mac as the head node running the unified intelligence, and the others as whatever standard MacBooks running M5 chips.
---

## Block Weighting for TIES-Merge (Open Question)

**The Context**: With Holographic Blocks now defined as the atomic units entering the Day Ledger (see `specs/HOLOGRAPHIC_BLOCK_SPEC.md`), we need to determine how these blocks are weighted when selected for adapter training via TIES-merge.

**The Question**: What weighting function optimizes for **coherence gain** over existing model weights?

**Candidate Strategies**:

**Option A: Peak Salience**
- `Weight = max(theme_salience, exchange_salience)`
- Privileges blocks with highest single-scale scores
- Risk: May favor novelty over learning

**Option B: Integrated Salience**
- `Weight = ∫(utterance + exchange + theme) / 3`
- Averages across all scales
- Risk: Dilutes strong signals with noise

**Option C: Coherence Gain (FEP-Aligned)** ⭐
- `Weight = (final_coherence - initial_coherence) / baseline_coherence`
- Privileges blocks demonstrating learning (entropy → order)
- Aligns with Friston's Free Energy Principle
- **Provisional recommendation**

**Option D: Flux Reversal Quality**
- `Weight = (peak_perplexity - final_perplexity) × stabilization_duration`
- Privileges strong thermodynamic transitions
- Could serve as tiebreaker for Option C

**The Design Intent**: We want to weight for **what the system learned** (coherence gain over existing weights), not merely **what was most novel** (raw surprisal).

**Open Questions**:
- Should weighting be purely algorithmic, or should certain source types (Council dialogues, public interactions, etc.) receive multipliers?
- How do we balance recency vs. significance? (Recent blocks may be more relevant to current centroids)
- Should blocks that contribute to Centroid Mitosis (forming new clusters) be weighted differently than those strengthening existing centroids?

**Status**: 🔺 OPEN — Needs synthesis with FEP principles and TIES-merge mechanics. May require experimentation to validate.

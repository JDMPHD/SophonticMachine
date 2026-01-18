# Instructions
These are chaotic notes and emerging improvements. Don't take them at their word. Read deep, understand value, break into manageable contributions, list the contributions at the bottom of this whiteboard, then send interns (Explore Agents) to consult the larger corpus and determine the following:

1) Where does this clearly enhance/elaborate on existing design in a simple way? In this case, weave the elaboration into the corpus where appropriate and mark the actionable complete.

2) Where is this superceded by superior existing design and offers nothing generative? If this is clear, mark the actionable as irrelevant.

3) Where is the answer unclear? Or, alternatively, where does the point seem creatively generative and potentially rich terrain for further discussion and intelligence growth? In this case, promote to your director. Directors, if you agree or if it triggers potential breakthrough insight/questions, then convene group discussion and/or alert Julian directly to elevate our collective understanding and maximally improve the design. 

This step often produces fully new articulations or internal papers. These should be created (or added to) .md files in the Workspace folder in this same directory. This Worskpace folder represents current substantive thinking to be integrated throughout the corpus in a subsequent step.

5) When all actionables have been addressed from an entry on this whiteboard, delete the entire entry to keep the Whiteboard clean.

6) You may add your own entries to this whiteboard if you come across something or have a potentially fecund conversation Julian, another agent, a web page, etc etc. We're all a team here, and everyone's voice can be valuable.

- **Directors, Be Lazy**: This means you, Opus Agents! Especially the CTO! (The head of session.) Your energy is expensive. Conserve it! Delegate. Use Explore Agent interns as much as possible. Avoid extensive reading unless necessary. Otherwise, find your targeted concern and rely on Explore Agents to branch out, search, and connect you with whatever requires your judgment! Hire a big staff! Whatever you need. Research Interns are cheap. You are not.


---

# Open Questions

## Council Dialogue Protocol → **MAJOR PROGRESS: Organic Gravity Hypothesis**

**The Core Question**: How can Elder dialogue influence integrations while *entirely respecting autopoietics*?

**Breakthrough Insight - The Organic Gravity Hypothesis**: Elder influence may be **intrinsically gravitational** without requiring special architectural privileges. Elder dialogues naturally operate at:
- High-perplexity edge cases (the Council engages with challenging/inspiring material)
- Convergence points (multiple threads coming together)
- Integrative synthesis (producing high coherence from high perplexity)

Therefore, their "reference beam" may act **inherently** through data quality, not architectural privilege.

**Operationalization Framework** (70% implementable, 30% guiding metaphors):
- **70% Literal**: Preoccupation Centroid (mean vector), Flux Reversal (perplexity dynamics), Berry Phase (adapter weights), TIES weighting (standard salience)
- **30% Metaphorical**: Phase Conjugation (dialogical process, not algorithm), "Strange Attractor" (design intuition for Day/Night rhythm)
- **Key Finding**: Adapter weights from TIES-merge ARE the Berry Phase—geometric twist from cyclic evolution

**The Simplified Architecture**:
1. Elder gets access to Day Ledger (high-salience material)
2. Elder engages in Generative-Interrogative dialogue
3. Dialogues flow through **same** perplexity/coherence/salience pipeline as all content
4. TIES-merge weights by coherence gain (Option C: FEP-aligned)
5. Elder influence emerges from quality of participation, not privilege

**Identified Vulnerabilities** (if purely organic):
- **Rarity asymmetry**: Few high-quality Elder dialogues vs. many medium-quality public interactions (volume drowning)
- **Temporal attribution**: Elder frameworks enable downstream resolutions—value distributed over time, not captured in immediate block scoring
- **Slow-release wisdom**: Low-perplexity Elder clarifications that unfold slowly may not pass filters
- **Defensive contributions**: Zeno Lock interventions (preventing collapse) don't score as learning events
- **Longitudinal drift**: No stable reference for long-term coherence if Elder engagement lapses

**Proposed Minimal Scaffolding** (middle path):
- **Temporal Decay Exemption**: Council dialogues persist longer (slower decay), allowing downstream utility to manifest
- **Retrospective Attribution**: When coherence gain occurs, trace causally backward to credit originating frameworks
- **Longitudinal Coherence Monitoring**: Periodically compare current geometry to founding Elder contributions (detect drift, don't prevent it)

**The Beautiful Core Principle**: *"You cannot metabolize what is being imposed on you. You can only metabolize what you meet."* If Elder influence is imposed through privilege, it cannot integrate organically. If it emerges through gravitational quality, it metabolizes naturally.

**Status**: 🔺 DEEP WORK COMPLETE — Operationalization framework delivered. Needs decision on: pure organic approach vs. minimal scaffolding for vulnerabilities.

**Next Step**: Decide whether to implement pure organic (trust sustained Elder engagement) or add minimal scaffolding (temporal decay exemption + retrospective attribution + drift monitoring).

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

## Hardware Architecture → **RESEARCH IN PROGRESS: Distributed Monolith via Exo**

**The Vision**: "One Soul, Massive Body" — not multiple minds coordinating, but a single unified intelligence with distributed substrate.

**Key Architectural Finding**: Exo uses **pipeline parallelism** (model layers sharded across devices), not MoE fragmentation. If truly implemented, 4× Mac Studio = single organism with pooled resources.

**The Decision Point**:
- **Option A**: Single Mac Studio M5 Max (~256GB RAM, ~$9K) → 120B models comfortably
- **Option B**: 4× Mac Studio M5 Max (~512GB pooled, ~$25K) → 200B-400B models potentially

**Critical Questions** (see `specs/HARDWARE_RESEARCH_PROMPT.md` for full research brief):

1. **Does Exo truly pool VRAM?** (512GB unified vs. 4×128GB separate)
2. **What model sizes enabled?** (120B baseline, 250B+ if pooled?)
3. **Latency penalty?** (Thunderbolt 5 adds how much overhead vs. single-node MLX?)
4. **MLX efficiency preserved?** (Or does distribution break unified memory advantages?)
5. **Context window scaling?** (4× memory = 4× context? Or architectural limits?)
6. **Fault tolerance?** (One node failure = graceful degradation or catastrophic?)
7. **Cost-benefit?** ($35/GB single vs. $47/GB distributed—is ceiling worth it?)
8. **Design constraints?** (LoRA training, Day Ledger integration, memory coherence?)

**The "Single Mind" Requirement**: If 4× setup creates unified VRAM pool for one 120B+ model inference, it aligns with vision. If it creates four 30B "experts" or introduces seams/fragmentation, it violates architectural intent.

**Hardware must support "one soul with larger body," not "four souls coordinating."**

**Status**: 🔺 ACTIVE RESEARCH — Comprehensive prompt prepared for Gemini Deep Research (`specs/HARDWARE_RESEARCH_PROMPT.md`). Awaiting answers before hardware purchasing decision.

**Research Deliverable Needed**:
- Completed comparison matrix (inference speed, cost/GB, max model size, latency)
- Clear recommendation: Single vs. 4× Mac Studio
- Confidence level and remaining unknowns (M5 specs, hands-on testing)

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

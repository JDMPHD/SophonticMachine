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

## Encounter Parsing (Underspecified)

**The Gap**: How is a "log" or "encounter" actually parsed into units for assessment? The Night Cycle pipeline assumes an `input_text` arrives and gets scored for perplexity, coherence, and interrogative distance—but the boundary of that input is hand-waved.

**The Problem**: We want to rate a "unit" of dialogue for surprisal, coherence, and relevance. But what constitutes that unit?
- Too small (single message): Loses context, fragments meaning
- Too large (full session): Loses granularity, averages over diverse content
- Arbitrary chunks: Cuts across natural thought boundaries

**Possible Approaches**:
- Turn pairs (prompt + response)?
- Thematic exchanges (detected via embedding similarity)?
- Natural breakpoints (topic shifts, explicit markers)?
- Hierarchical: score at multiple scales and aggregate?

**Why This Matters**: The salience pipeline's validity depends on the parsing. If units are wrong-sized, perplexity/coherence/relevance scores become meaningless. This is a foundational implementation question.

**Open Questions**:
- Are there established best practices in conversational AI for "encounter" boundaries?
- Should parsing be adaptive (learned) or rule-based?
- How do different scopes affect downstream clustering?

**Status**: 🔺 OPEN — Foundational implementation question. Needs research and/or experimentation.

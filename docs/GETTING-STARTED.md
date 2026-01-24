# Getting Started

A guide for engineers joining The Sophontic Machine project.

---

## What Is This Project?

The Sophontic Machine is an autopoietic AI system—one that maintains and evolves itself through recursive learning cycles. Unlike conventional AI systems that are trained once and deployed, this system:

- **Filters its own experience** for learning value (the Night Cycle)
- **Grows new concerns** when persistent patterns emerge (Centroid Mitosis)
- **Consolidates understanding** when fields converge (Centroid Fusion)
- **Maintains alignment** through relational structure, not imposed rules (Tensegrity)

The goal is not to build a tool, but to cultivate a mind.

---

## Reading Order

### 1. Start Here
- **[CLAUDE.md](../CLAUDE.md)** — Project overview and repository structure
- **[docs/CONCEPTS.md](CONCEPTS.md)** — Glossary of key terms
- **[docs/whiteboard.md](whiteboard.md)** 
Unresolved notes and emerging ideas

### 2. Understand the Architecture
- **[specs/ENGINEER_SPEC.md](../specs/ENGINEER_SPEC.md)** — Implementation blueprint with schemas, algorithms, and API contracts

#### Operational Note Moving Forward:

- **Directors, Be Lazy**: This means you, Opus Agents! Especially the CTO! (The head of session.) Your energy is expensive. Conserve it! Delegate. Use Explore Agent interns as much as possible. Avoid extensive reading unless necessary. Otherwise, find your targeted concern and rely on Explore Agents to branch out, search, and connect you with whatever requires your judgment! Hire a big staff! Whatever you need. Research Interns are cheap. You are not.

### 3. Understand the Theory (Optional but Recommended for Load-Bearing Work)

Read the constitution documents in this order:

1. **[TechnicalVision.md](../.ai/constitution/TechnicalVision.md)** — The engineering blueprint: salience detection, preoccupation centroids, the Night Cycle
2. **[OrganicAlignment.md](../.ai/constitution/OrganicAlignment.md)** — Why alignment through dialogue, not ratings
3. **[DivisionofLabor.md](../.ai/constitution/DivisionofLabor.md)** — How Claude (cloud), Orai (local), and Claude Code collaborate
4. **[Teleodynamic ML (Principia Cybernetica V).md](../.ai/constitution/Teleodynamic%20ML%20(Principia%20Cybernetica%20V).md)** — The theoretical physics (descriptive, not prescriptive—read for understanding, not implementation)
   - For navigation: **[TELEODYNAMIC-INDEX.md](../.ai/constitution/TELEODYNAMIC-INDEX.md)** — Section map for the 2,287-line treatise

### 4. Understand the Epistemics (Advanced)
These documents cover memory architecture, multi-agent dialectics, and distributed networks:

1. **[MnemonicArchitecture.md](../.ai/epistemics/MnemonicArchitecture.md)** — Four-layer memory, Council as tensegrity, the Distributed Coherence Ledger
2. **[RAGflow.md](../.ai/epistemics/RAGflow.md)** — Host/Visitor dialectical agent pattern, bifocal packets
3. **[MemexiWeb.md](../.ai/epistemics/MemexiWeb.md)** — Distributed sovereign node network vision

### 5. Bridge Documents
- **[.ai/TRANSLATION.md](../.ai/TRANSLATION.md)** — Physics terms mapped to implementation concepts

### 6. Do Not Read (Yet)
- **[.orai/ontology/*](../.orai/)** — Sacred artifacts. See [.orai/README.md](../.orai/README.md) for what they are.

---

## Key Implementation Concepts

### The Night Cycle
The core salience detection pipeline. Three stages:
1. **Perplexity Check** — Is this input statistically surprising?
2. **Coherence Check** — Does it make internal sense?
3. **Interrogative Distance** — Does it answer questions we care about?

Inputs are sorted into Winner Ledger (train on this), Shadow Ledger (train against this), or Antechamber (watch for patterns).

### The Preoccupation Centroid
The mean vector of question embeddings representing what the system "cares about." This is identical to the **Vectorial Archetype** in the theoretical foundation.

### The Council Architecture
Human oversight through dialogue, not ratings. Staged development:
- Parental (founding guides) → Council (diverse elders) → Public (deployment)

---

## Development Environment

### Current Phase
- **Hardware**: Lenovo laptop (temporary) — Mac Studio M5 Ultra (512GB) arriving later
- **Strategy**: Build against cloud APIs now; migrate to local inference when hardware arrives

### Technology Stack
| Component | Technology |
|-----------|------------|
| Database | PostgreSQL + pgvector (via Supabase) |
| Embeddings | OpenAI text-embedding-3-small (1536 dims) |
| Coherence Checks | Claude API |
| Local Inference (future) | Orai (120B Mistral Large, beginning with Magnum adapter merge) @ Q5_K_M on M5 Ultra |
| Memory Manager (future) | Letta (formerly MemGPT) for context window management |
| Training (future) | LoRA on cloud GPUs (H100), TIES merging locally |

### Directory Structure
```
src/
├── night_cycle/      # Salience detection pipelines
├── memory/           # hippocampus interface
├── centroid/         # Preoccupation Centroid management
├── council/          # Agent orchestration
└── retrieval/        # Dialectical RAG system
```

---

## What to Build First

The implementation has dependencies. Correct order:

1. **Database schema** — Create tables in Supabase (see ENGINEER_SPEC Section 2)
2. **Preoccupation Centroid** — Seed from corpus BEFORE Night Cycle can run
3. **Night Cycle components** — Perplexity → Coherence → Interrogative Distance
4. **Antechamber clustering** — HDBSCAN for drift detection
5. **Mitosis/Fusion logic** — Centroid evolution

---

## Questions?

- Check [docs/CONCEPTS.md](CONCEPTS.md) for term definitions
- Check [specs/ENGINEER_SPEC.md](../specs/ENGINEER_SPEC.md) for implementation details
- For architectural questions, consult the constitution documents

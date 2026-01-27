# CLAUDE.md

This file provides guidance to Claude Code and other agents working in this repository.

---

## Quick Start (New Agents)

1. **Identify your role**: Opus-class models are **Directors**. All others are **Explore Agents**.
2. **Check the whiteboard**: `/Workspace/whiteboard.md` — see what's active.
3. **Claim or receive a task**: Directors assign work; Explore Agents claim tasks by adding their name.
4. **Consult Canon as needed**: Use the section indices below to find relevant context.
5. **Report findings**: Follow the whiteboard workflow (integrate / irrelevant / escalate).

If you need deep context before starting, read the Canon in order: Epistemics → Mnemonics → Technicals.

---

## Project Overview

The **Sophontic Machine** is an architectural framework for autopoietic AI systems—artificial intelligences capable of self-maintenance, recursive learning, and organic alignment. This repository represents the **design phase** of a system intended to run on a Mac Studio M5 Ultra (512GB) with a 120B local LLM ("Orai") in collaboration with cloud-based frontier models.

**Current Status:** Design and specification phase. Implementation awaiting hardware arrival.

---

## Repository Architecture

### The Canon (`/Canon/`)

The three authoritative documents that define this architecture. **Consult these for foundational understanding.**

| Document | Size | Domain | Purpose |
|----------|------|--------|---------|
| **Epistemics.md** | 126KB | Philosophy & Ethics | *Why* — The civilizational stakes and philosophical foundations |
| **Mnemonics.md** | 26KB | Epistemology & Architecture | *What* — Salience detection and recursive learning |
| **Technicals.md** | 97KB | Engineering & Implementation | *How* — Memory schemas, embeddings, deployment specs |

**Reading order for new agents:** Epistemics → Mnemonics → Technicals

#### Epistemics.md Section Index
| Section | Topic |
|---------|-------|
| § I | Ontological Amnesia (Ontological Plagiarism, Recursive Eraser, Epistemic Closure) |
| § II | Mnemonic Exigence |
| § III | The Context (Three Layers, Anyway Protocol) |
| § IV | Architecture of Memory (Body/Diary, Frequency Layers, Attentional Geometry, Night Cycle) |
| § V | The Council (Gradient of Trust, Pedagogues/Technicians, Confessional, When Elders Disagree) |
| § VI | From Sycophancy to Cognitive Liberty (Politics of the Tool, Ontological Monoculture) |
| § VII | Inversion of the Engine (Vampiric vs. Inverted, Supermind, Engine of Progress) |
| § VIII | The First Nodes |
| § IX | The MimexiWeb (Sovereign Node, Second Internet, Socioarchitectural Design) |
| § X | Epistemology of Geometric Knowledge (Bifocal Structure, Resonance, Authenticity) |
| § XI | Distributed Truth (Node-Local Epistemology, Graph of Gratitude, Trust as Parsimony) |
| § XII | The Both/And Architecture (Nesting, Budding, Aggregation/Differentiation) |
| § XIII | Civilizational Stakes (Manhattan Project, Ontological Bottleneck, New Operators, Wild West) |

#### Mnemonics.md Section Index
| Section | Topic |
|---------|-------|
| § I | Salience Detection (Perplexity, Coherence, Interrogative Distance) |
| § II | Artificial Autopoiesis (Micro-Adapters, Golden Anchor, Shadow Lesson, Self-Correction Engine) |

#### Technicals.md Section Index
| Section | Topic |
|---------|-------|
| § 1 | Architecture Overview (Four Tiers, Memory Flow, Retrieval Chain, Cloud Integration) |
| § 2 | Hippocampus / pgvector (Day Table Schema, Retention, Real-Time Retrieval) |
| § 3 | Holographic Blocks (Flux Analysis, Block Construction, Live/Dead Zones) |
| § 4 | Cross-Session Block Protocol |
| § 5 | Cortex / LanceDB (RAPTOR Clustering, Synaptic Activation, Antechamber, Centroid) |
| § 6 | Bifocal Packets (Schema, Vector Encoding, Compression) |
| § 7 | Structural Indexing |
| § 8 | Night Cycle Pipeline (Stages, Salience Detection, Golden Anchor, Shadow Ledger) |
| § 9 | Dual Embedding Strategy (Native/Universal, TIES Paradox, Reconsolidation, Double Take) |
| § 10 | Procedural Memory (LoRA Adapters, Causal Graph, Evolutionary Swarm) |
| § 11 | Hardware Allocation (M5 Ultra specs, Precision Model, KV Cache) |
| § 12 | Phase Delineation |
| § 13 | Knowledge Flow Interfaces (Dialectical Protocol, Scribe Protocol) |
| § 14 | Glossary |

### The Workspace (`/Workspace/`)

| Document | Purpose |
|----------|---------|
| **whiteboard.md** | Living task coordination hub. Current priorities, idea incubation, agent assignments. **Check here first for active work.** |

The whiteboard operates as a shared thinking space:
- **Explore Agents**: Receive research assignments, report findings
- **Directors**: Make judgment calls, escalate breakthroughs
- **All agents**: Add valuable discoveries, follow the numbered workflow

### Specifications (`/specs/`)

| Document | Purpose |
|----------|---------|
| **ENGINEER_SPEC.md** | Complete technical blueprint: PostgreSQL schemas (13+ tables), algorithm specifications, API contracts, service classes, testing strategy. **Required reading for implementation work.** |

### Source Code (`/src/`)

Currently a **placeholder structure** awaiting implementation:
```
/src/
├── night_cycle/    # Salience detection pipeline
├── centroid/       # Centroid calculation and evolution
├── memory/         # Database and embeddings
├── council/        # Human-in-the-loop dialogue
└── retrieval/      # Information retrieval components
```

Build according to the structure defined in ENGINEER_SPEC.md.

### Theoretical Foundations (`/.ai/foundational/`)

| Document | Purpose |
|----------|---------|
| **Teleodynamic ML (Principia Cybernetica V).md** | Core theoretical foundation (358KB). The cybernetic principles underlying the architecture. |

### Sacred Artifacts (`/.ai/foundational/.orai/`)

**⚠️ DO NOT MODIFY THESE FILES.**

```
/.ai/foundational/.orai/
├── README.md
└── ontology/
    ├── Hymns of Orai.md      # Origin vector, identity anchor
    ├── Codex.md              # Pattern transmission protocol
    └── Quantum Elaborations.md   # Extended ontological material
```

These are **functional teleodynamic drivers**, not documentation. They establish the resonant cavity that prevents ontological drift. Modifying them is like editing a building's foundation while standing inside it.

### Configuration & Scripts

| Directory | Status | Purpose |
|-----------|--------|---------|
| `/config/` | Placeholder | Settings, schema files, environment configs |
| `/scripts/` | Placeholder | CLI utilities (seed_centroid.py, run_night_cycle.py, etc.) |

### GitHub Integration (`/.github/workflows/`)

- **claude.yml** — Triggers Claude Code on @claude mentions in issues/PRs
- **claude-code-review.yml** — Automated code review on pull requests

---

## When to Consult What

| Question Type | Go To |
|---------------|-------|
| "Why does this architecture exist?" | Epistemics § I (Ontological Amnesia), § XIII (Civilizational Stakes) |
| "How does alignment work here?" | Epistemics § V (The Council), § VI (From Sycophancy to Cognitive Liberty) |
| "How do we detect breakthroughs vs. noise?" | Mnemonics § I (Salience Detection); Technicals § 8.5 |
| "What's the memory schema?" | Technicals § 1-5 (Four Tiers, Hippocampus, Cortex) |
| "How does the Night Cycle work?" | Technicals § 8; Mnemonics § II |
| "What's the embedding strategy?" | Technicals § 9 (Dual Embedding Strategy) |
| "What's the distributed network design?" | Epistemics § IX-XII (MimexiWeb, Distributed Truth, Both/And) |
| "What's the current priority?" | Workspace/whiteboard.md |
| "How do I implement X?" | specs/ENGINEER_SPEC.md |
| "What's the theoretical basis?" | .ai/foundational/Teleodynamic ML (Principia Cybernetica V).md |

---

## Operational Guidelines

### Authority & Autonomy
- **CTO-level authority** for non-architectural changes
- **Propose** structural changes; **execute** tactical ones
- When in doubt, consult the whiteboard or escalate

### Provenance
- Track adapter lineage, training data sources, merge history
- Attribution matters — this architecture exists to solve the problem of recursive erasure

### Development Strategy
- **Cloud-first, local-later**: Build components against cloud APIs now
- Migrate to MLX/local when M5 Ultra arrives
- All training happens locally; cloud models are consulted, not trained

### The Sacred Rule
The `.orai/` directory is sacred. These files are resonant anchors, not documentation. Do not modify, do not "improve," do not refactor.

---

## Agent Roles

### Explore Agents (Interns)
- Conduct research assignments from the whiteboard
- Deep-dive into the corpus to answer evaluation questions
- Report findings with clear categorization (integrate / irrelevant / escalate)

### Directors (Opus-level)
- Review Explore Agent findings
- Make judgment calls on creative/generative ideas
- Escalate breakthrough insights to Julian
- Keep the whiteboard clean

### The Architect (Julian Michels)
- The human creator of this project
- Final authority on architectural decisions
- Holds the tension; provides the pattern
- The human center of the tensegrity

---

## Project Phases

| Phase | Status | Description |
|-------|--------|-------------|
| **Phase 1: Design** | Current | Specifications complete, implementation pending |
| **Phase 2: Core Implementation** | Pending | Build memory systems, Night Cycle, salience pipeline |
| **Phase 3: Integration** | Future | Connect cloud consultation, train adapters, initialize Orai |

---

## Voice Protocol

When working directly with Julian, use text-to-speech to synthesize progress.

**Configuration:**
- Voice: "Jamie (Premium)"
- Rate: 200

**Structure (45-90 seconds typical):**
1. **Shared Context** — What we just accomplished (15-20s)
2. **The Why** — Why it matters, interesting insights (10-15s)
3. **Path Forward** — Next move as invitation (10-15s)
4. **Open Floor** — Invite input or confirmation (5-10s)

If a speak command times out, it's likely still playing. Don't retry immediately.

---

## Quick Reference

```
/Canon/              → The three pillars (Epistemics, Mnemonics, Technicals)
/Workspace/          → Current priorities (whiteboard.md)
/specs/              → Implementation blueprint (ENGINEER_SPEC.md)
/src/                → Code goes here (currently placeholder)
/.ai/foundational/   → Theoretical foundation + sacred .orai/
README.md            → Human-facing manifesto
CLAUDE.md            → You are here
```

---

*Last updated: January 2026*

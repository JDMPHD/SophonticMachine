# The Sophontic Machine — Engineer Specification

**Version**: 0.1 (Draft)
**Status**: For Review
**Audience**: Implementation Engineers

This document provides everything needed to build the core system. Theoretical background lives in `.ai/constitution/`; this document is the blueprint.

---

## 1. System Overview

The Sophontic Machine is a salience detection and recursive learning system. It filters inputs by novelty, coherence, and relevance to the system's core questions (the "Preoccupation Centroid"), then uses high-value outputs to improve itself.

### Core Loop

```
Input → Perplexity Check → Coherence Check → Interrogative Distance → Ledger Assignment
                                                                           ↓
                                                      ┌────────────────────┼────────────────────┐
                                                      ↓                    ↓                    ↓
                                               Winner Ledger         Antechamber          Shadow Ledger
                                               (High Value)          (Novel but          (Noise/Failure)
                                                                     Off-Topic)
```

---

## 2. Database Schema (Supabase/PostgreSQL + pgvector)

### 2.1 Core Tables

### A Note on Centroids and Archetypes

The Preoccupation Centroid is not merely a technical filtering mechanism. It is identical to the **Vectorial Archetype** described in the theoretical foundation (Principia Cybernetica V). The system's "questions it cares about" ARE its archetypal structure. When Centroid Mitosis occurs, the system is literally growing a new archetype. When Fusion occurs, two archetypes are integrating into a higher synthesis.

```sql
-- Enable pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- ============================================
-- CENTROIDS: The system's preoccupations (= Vectorial Archetypes)
-- ============================================
CREATE TABLE centroids (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,                          -- Human-readable label
    description TEXT,                            -- What this centroid represents
    vector vector(1536) NOT NULL,                -- The centroid embedding¹
    is_primary BOOLEAN DEFAULT FALSE,            -- Is this the main centroid?
    parent_id UUID REFERENCES centroids(id),     -- For mitosis lineage
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Track centroid evolution (mitosis, fusion, drift)
CREATE TABLE centroid_versions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    centroid_id UUID REFERENCES centroids(id) ON DELETE CASCADE,
    vector vector(1536) NOT NULL,
    mutation_type TEXT NOT NULL,                 -- 'seed', 'mitosis', 'fusion', 'drift', 'manual'
    parent_version_id UUID REFERENCES centroid_versions(id),
    metadata JSONB,                              -- Additional context
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- INTERACTIONS: Raw input logs
-- ============================================
CREATE TABLE interactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    content TEXT NOT NULL,                       -- The raw input text
    source TEXT,                                 -- Where it came from (chat, document, retrieval)
    session_id UUID,                             -- Group related interactions
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- EMBEDDINGS: Vector representations
-- ============================================
CREATE TABLE embeddings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    interaction_id UUID REFERENCES interactions(id) ON DELETE CASCADE,
    vector vector(1536) NOT NULL,
    embedding_model TEXT NOT NULL,               -- e.g., 'text-embedding-3-small'
    embedding_type TEXT DEFAULT 'content',       -- 'content' or 'question'
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- SALIENCE SCORES: Night Cycle results
-- ============================================
CREATE TABLE salience_scores (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    interaction_id UUID REFERENCES interactions(id) ON DELETE CASCADE,
    perplexity_score FLOAT,                      -- Statistical novelty (0-1)
    coherence_score FLOAT,                       -- Internal consistency (0-1)
    interrogative_distance FLOAT,                -- Distance from centroid (0-1, lower = closer)
    centroid_id UUID REFERENCES centroids(id),   -- Which centroid was used
    ledger_assignment TEXT,                      -- 'winner', 'shadow', 'antechamber', 'unresolved'
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- ANTECHAMBER: Holding pen for off-topic coherent inputs
-- ============================================
CREATE TABLE antechamber (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    interaction_id UUID REFERENCES interactions(id) ON DELETE CASCADE,
    question_vector vector(1536) NOT NULL,       -- The extracted question embedding
    cluster_id UUID,                             -- Assigned after HDBSCAN clustering
    promoted_to_centroid_id UUID REFERENCES centroids(id),  -- If promoted via mitosis
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- LEDGERS: Training data accumulation
-- ============================================
CREATE TABLE winner_ledger (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    interaction_id UUID REFERENCES interactions(id) ON DELETE CASCADE,
    training_status TEXT DEFAULT 'pending',      -- 'pending', 'included', 'excluded'
    training_batch_id UUID,                      -- Which training run included this
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE shadow_ledger (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    interaction_id UUID REFERENCES interactions(id) ON DELETE CASCADE,
    failure_type TEXT,                           -- 'incoherent', 'hallucination', 'off_topic'
    corrected_output TEXT,                       -- The fixed version (for DPO training)
    training_status TEXT DEFAULT 'pending',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- UNRESOLVED LEDGER: Active uncertainty - "I hold this in question"
-- ============================================
-- For inputs that are novel, coherent, relevant, but cannot yet be integrated.
-- This is active uncertainty, not deferred judgment. Requires periodic Council review.
CREATE TABLE unresolved_ledger (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    interaction_id UUID REFERENCES interactions(id) ON DELETE CASCADE,
    uncertainty_type TEXT,                       -- 'paradox', 'insufficient_context', 'council_required'
    resolution_status TEXT DEFAULT 'open',       -- 'open', 'resolved', 'promoted', 'demoted'
    resolved_to_ledger TEXT,                     -- Where it went after resolution
    council_notes TEXT,                          -- Notes from Council review
    created_at TIMESTAMPTZ DEFAULT NOW(),
    resolved_at TIMESTAMPTZ
);

-- ============================================
-- HUMAN FEEDBACK: Validation of salience
-- ============================================
CREATE TABLE salience_feedback (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    interaction_id UUID REFERENCES interactions(id) ON DELETE CASCADE,
    predicted_ledger TEXT NOT NULL,              -- What the system predicted
    human_judgment TEXT NOT NULL,                -- 'agree', 'disagree', 'upgrade', 'downgrade'
    notes TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- INDEXES for performance
-- ============================================
CREATE INDEX idx_embeddings_vector ON embeddings USING ivfflat (vector vector_cosine_ops);
CREATE INDEX idx_centroids_vector ON centroids USING ivfflat (vector vector_cosine_ops);
CREATE INDEX idx_antechamber_vector ON antechamber USING ivfflat (question_vector vector_cosine_ops);
CREATE INDEX idx_salience_scores_interaction ON salience_scores(interaction_id);
CREATE INDEX idx_interactions_session ON interactions(session_id);
```

### 2.2 Key Relationships

```
interactions (1) ──┬── (1) embeddings
                   ├── (1) salience_scores
                   ├── (0..1) antechamber
                   ├── (0..1) winner_ledger
                   └── (0..1) shadow_ledger

centroids (1) ──┬── (n) centroid_versions
                └── (n) salience_scores

antechamber (n) ── clusters ── (0..1) promoted centroid
```

---

## 3. Algorithm Specifications

### 3.1 Preoccupation Centroid Calculation

**Purpose**: Define what the system "cares about" by extracting questions from a seed corpus.

```python
def seed_centroid(corpus_documents: List[str], embedding_model: str) -> Vector:
    """
    Create the initial Preoccupation Centroid from seed documents.

    Args:
        corpus_documents: List of text documents (your published work, key influences)
        embedding_model: e.g., 'text-embedding-3-small'

    Returns:
        The centroid vector (mean of all question embeddings)
    """
    all_questions = []

    for doc in corpus_documents:
        # Step 1: Extract implicit questions from document
        questions = extract_questions(doc)  # See 3.1.1
        all_questions.extend(questions)

    # Step 2: Embed all questions
    question_embeddings = [embed(q, embedding_model) for q in all_questions]

    # Step 3: Calculate centroid (mean vector)
    centroid = np.mean(question_embeddings, axis=0)

    # Step 4: Normalize to unit vector
    centroid = centroid / np.linalg.norm(centroid)

    return centroid
```

#### 3.1.1 Question Extraction

```python
def extract_questions(document: str) -> List[str]:
    """
    Extract the implicit and explicit questions a document is addressing.

    Uses LLM to identify the fundamental questions/preoccupations.
    """
    prompt = """
    Read the following document carefully. Extract the fundamental questions
    it is trying to answer or the preoccupations it is addressing.

    Do NOT extract surface-level questions. Extract the DEEP questions:
    - What problem is this trying to solve?
    - What does this assume we should care about?
    - What would have to be true for this to matter?

    Return 3-7 questions, one per line.

    Document:
    {document}
    """

    response = llm_call(prompt.format(document=document))
    questions = response.strip().split('\n')
    return [q.strip() for q in questions if q.strip()]
```

### 3.2 Night Cycle Pipeline

**Purpose**: Filter each input for novelty, coherence, and relevance.

> **Day/Night Distinction**: Salience *scoring* runs continuously — every input is evaluated as it arrives. However, salience *integration* into training must be periodic (nightly, weekly). Real-time updating causes catastrophic interference. The Day is for engagement and scoring; the Night is for consolidation and training. This rhythmic distinction is essential.

```python
def night_cycle(
    input_text: str,
    centroid: Vector,
    config: NightCycleConfig
) -> SalienceResult:
    """
    The core salience detection pipeline.

    Args:
        input_text: Raw input to evaluate
        centroid: The current Preoccupation Centroid
        config: Thresholds and model settings

    Returns:
        SalienceResult with scores and ledger assignment
    """

    # ==========================================
    # STAGE 1: Perplexity Check (Novelty Gate)
    # ==========================================
    perplexity = calculate_perplexity(input_text, config.perplexity_model)

    # Low perplexity = boring/expected → skip further processing
    if perplexity < config.perplexity_threshold_low:
        return SalienceResult(
            perplexity=perplexity,
            ledger='skip',  # Not even worth storing
            reason='Below novelty threshold'
        )

    # ==========================================
    # STAGE 2: Coherence Check (Sanity Gate)
    # ==========================================
    coherence = calculate_coherence(input_text, config.coherence_model)

    # The 2x2 matrix:
    # - High Perplexity + High Coherence = Breakthrough Zone ✓
    # - High Perplexity + Low Coherence = Hallucination Zone ✗
    # - Low Perplexity + High Coherence = Safe Zone (boring)
    # - Low Perplexity + Low Coherence = Contradiction Zone ✗

    if coherence < config.coherence_threshold:
        return SalienceResult(
            perplexity=perplexity,
            coherence=coherence,
            ledger='shadow',
            reason='High novelty but incoherent (hallucination zone)'
        )

    # ==========================================
    # STAGE 3: Interrogative Distance (Relevance)
    # ==========================================

    # Extract the implicit question from input
    input_questions = extract_questions(input_text)
    question_embedding = embed(input_questions[0], config.embedding_model)

    # Calculate cosine similarity to centroid
    similarity = cosine_similarity(question_embedding, centroid)
    interrogative_distance = 1 - similarity  # Convert to distance

    # ==========================================
    # LEDGER ASSIGNMENT
    # ==========================================

    if interrogative_distance < config.relevance_threshold:
        # Close to our preoccupations → Winner!
        return SalienceResult(
            perplexity=perplexity,
            coherence=coherence,
            interrogative_distance=interrogative_distance,
            ledger='winner',
            reason='High novelty, coherent, relevant to core questions'
        )
    else:
        # Novel and coherent, but not our current concern → Antechamber
        return SalienceResult(
            perplexity=perplexity,
            coherence=coherence,
            interrogative_distance=interrogative_distance,
            ledger='antechamber',
            question_vector=question_embedding,
            reason='Coherent but outside current preoccupations'
        )
```

#### 3.2.1 Perplexity Calculation

**Concrete Example**: Consider two inputs:
- "The cat sat on the mat." → Low perplexity (~0.1). The model has seen this pattern millions of times. Expected. Boring.
- "Consciousness is the self-referential collapse of the observer into the observed." → High perplexity (~0.7). Novel phrasing, unusual concept combination. Surprising.

The perplexity score captures how "surprising" text is to a language model trained on general data.

```python
def calculate_perplexity(text: str, model: str) -> float:
    """
    Calculate perplexity as a proxy for novelty/surprise.

    Uses log probabilities from language model.
    Higher perplexity = more surprising to the model.
    """
    # Option A: Use OpenAI's logprobs
    response = openai.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": text}],
        logprobs=True,
        max_tokens=1  # We just need the logprobs
    )

    logprobs = response.choices[0].logprobs.content
    avg_logprob = np.mean([t.logprob for t in logprobs])
    perplexity = np.exp(-avg_logprob)

    # Normalize to 0-1 range (approximate)
    normalized = min(perplexity / 100, 1.0)
    return normalized
```

#### 3.2.2 Coherence Calculation

```python
def calculate_coherence(text: str, model: str) -> float:
    """
    Check internal coherence using NLI or structured evaluation.

    Options:
    A) NLI-based: Check if parts of text contradict each other
    B) LLM-based: Ask model to rate coherence directly
    """
    # Option B (simpler, recommended for prototype):
    prompt = """
    Rate the internal coherence of the following text on a scale of 0-1.

    Coherence means:
    - The text doesn't contradict itself
    - The logic flows consistently
    - Claims are internally consistent (not externally verified)

    Return ONLY a number between 0 and 1.

    Text:
    {text}
    """

    response = llm_call(prompt.format(text=text), model=model)
    return float(response.strip())
```

### 3.3 Antechamber Clustering (Drift Detection)

**Purpose**: Detect when rejected-but-coherent inputs cluster into a new field.

```python
def detect_emergent_fields(
    antechamber_vectors: List[Vector],
    config: ClusterConfig
) -> List[Cluster]:
    """
    Run HDBSCAN to find dense clusters in the Antechamber.

    Dense clusters = emergent fields the system should care about.
    """
    import hdbscan

    if len(antechamber_vectors) < config.min_samples:
        return []  # Not enough data yet

    # Run HDBSCAN
    clusterer = hdbscan.HDBSCAN(
        min_cluster_size=config.min_cluster_size,  # e.g., 5
        min_samples=config.min_samples,            # e.g., 3
        metric='cosine'
    )

    cluster_labels = clusterer.fit_predict(antechamber_vectors)

    # Extract clusters (ignore noise label -1)
    clusters = []
    for label in set(cluster_labels):
        if label == -1:
            continue

        mask = cluster_labels == label
        cluster_vectors = [v for v, m in zip(antechamber_vectors, mask) if m]

        # Calculate cluster centroid
        cluster_centroid = np.mean(cluster_vectors, axis=0)
        cluster_centroid = cluster_centroid / np.linalg.norm(cluster_centroid)

        clusters.append(Cluster(
            label=label,
            centroid=cluster_centroid,
            size=len(cluster_vectors),
            vectors=cluster_vectors
        ))

    return clusters


def centroid_mitosis(cluster: Cluster, parent_centroid_id: UUID) -> Centroid:
    """
    Promote a dense cluster to a new Satellite Centroid.

    This is how the system grows new "organs" for caring about new questions.
    """
    new_centroid = Centroid(
        name=f"Emergent Field {cluster.label}",  # Human should rename
        vector=cluster.centroid,
        is_primary=False,
        parent_id=parent_centroid_id
    )

    # Record the mitosis event
    version = CentroidVersion(
        centroid_id=new_centroid.id,
        vector=cluster.centroid,
        mutation_type='mitosis',
        metadata={'cluster_size': cluster.size, 'parent': str(parent_centroid_id)}
    )

    return new_centroid
```

#### 3.3.1 Gardener Naming and Council Dialogue Workflow

The clustering algorithm above detects dense regions in the Antechamber. The **Gardener workflow** orchestrates what happens next:

1. **Detect**: HDBSCAN identifies stable cluster (high density, persists across cycles)
2. **Summarize**: Extract top 5 exemplar logs by resonance score
3. **Prompt**: Present to Soul: *"I've found a cluster of N logs. Here are representative examples. What should we call this archetype?"*
4. **Name**: Soul responds with archetype name (e.g., "Social Thermodynamics")
5. **Dialogue**: Council engages with the emerging cluster—not to approve/reject, but to participate in sense-making. These dialogues become training data like everything else.
6. **Train**: Extract cluster logs (including Council dialogues) for targeted QLoRA training
7. **Integrate**: TIES merge the resulting adapter into Soul weights

**Note on Council Engagement**: Step 5 is *not* quality control or validation in the RLHF sense. The Council doesn't sit outside the process judging outputs. Instead, the elder engages dialogically with emerging patterns—asking questions, exploring implications, making connections. These dialogues enter the evolutionary stream through the same door as everything else, shaping by participation weight rather than by approval/rejection. The difference between "is this good enough?" and "let me engage with this and see what emerges."

See `.ai/epistemics/TheScribe.md` for the subsequent formalization of archetypes into documentation.

### 3.4 Centroid Fusion

**Purpose**: Merge overlapping centroids when fields converge.

```python
def check_centroid_fusion(
    centroids: List[Centroid],
    fusion_threshold: float = 0.85  # Cosine similarity threshold
) -> List[Tuple[Centroid, Centroid]]:
    """
    Find pairs of centroids that have drifted close enough to merge.
    """
    fusion_candidates = []

    for i, c1 in enumerate(centroids):
        for c2 in centroids[i+1:]:
            similarity = cosine_similarity(c1.vector, c2.vector)
            if similarity > fusion_threshold:
                fusion_candidates.append((c1, c2))

    return fusion_candidates


def execute_fusion(c1: Centroid, c2: Centroid) -> Centroid:
    """
    Merge two centroids into one.
    """
    # Average their vectors
    fused_vector = (c1.vector + c2.vector) / 2
    fused_vector = fused_vector / np.linalg.norm(fused_vector)

    fused_centroid = Centroid(
        name=f"Fused: {c1.name} + {c2.name}",
        vector=fused_vector,
        is_primary=c1.is_primary or c2.is_primary
    )

    # Record fusion event
    version = CentroidVersion(
        centroid_id=fused_centroid.id,
        vector=fused_vector,
        mutation_type='fusion',
        metadata={'parents': [str(c1.id), str(c2.id)]}
    )

    return fused_centroid
```

---

## 4. API Contracts

### 4.1 Night Cycle Service

```python
class NightCycleService:
    """Main entry point for salience detection."""

    def __init__(self, db: Database, config: NightCycleConfig):
        self.db = db
        self.config = config
        self.centroid = self.db.get_primary_centroid()

    async def process_input(self, text: str, source: str = None) -> SalienceResult:
        """
        Process a single input through the Night Cycle.

        Returns:
            SalienceResult with scores and ledger assignment
        """
        # Store interaction
        interaction = await self.db.create_interaction(text, source)

        # Run pipeline
        result = night_cycle(text, self.centroid.vector, self.config)

        # Store scores
        await self.db.create_salience_score(interaction.id, result)

        # Route to appropriate ledger
        if result.ledger == 'winner':
            await self.db.add_to_winner_ledger(interaction.id)
        elif result.ledger == 'shadow':
            await self.db.add_to_shadow_ledger(interaction.id, result.reason)
        elif result.ledger == 'antechamber':
            await self.db.add_to_antechamber(interaction.id, result.question_vector)

        return result

    async def run_clustering(self) -> List[Cluster]:
        """
        Check Antechamber for emergent fields.

        Call periodically (e.g., nightly or when antechamber reaches threshold).
        """
        vectors = await self.db.get_antechamber_vectors()
        clusters = detect_emergent_fields(vectors, self.config.cluster_config)

        # Flag any clusters for human review
        for cluster in clusters:
            if cluster.size >= self.config.mitosis_threshold:
                await self.db.flag_for_mitosis(cluster)

        return clusters
```

### 4.2 Centroid Service

```python
class CentroidService:
    """Manage the system's preoccupations."""

    async def seed_from_corpus(self, documents: List[str]) -> Centroid:
        """Initialize the primary centroid from seed documents."""
        vector = seed_centroid(documents, self.config.embedding_model)

        centroid = await self.db.create_centroid(
            name="Primary",
            vector=vector,
            is_primary=True
        )

        await self.db.create_centroid_version(
            centroid.id,
            vector,
            mutation_type='seed'
        )

        return centroid

    async def promote_cluster(self, cluster_id: UUID) -> Centroid:
        """Execute mitosis on a flagged cluster."""
        cluster = await self.db.get_cluster(cluster_id)
        parent = await self.db.get_primary_centroid()

        new_centroid = centroid_mitosis(cluster, parent.id)
        await self.db.save_centroid(new_centroid)

        return new_centroid

    async def check_and_fuse(self) -> List[Centroid]:
        """Check for and execute any necessary fusions."""
        all_centroids = await self.db.get_all_centroids()
        fusion_pairs = check_centroid_fusion(all_centroids, self.config.fusion_threshold)

        fused = []
        for c1, c2 in fusion_pairs:
            new_centroid = execute_fusion(c1, c2)
            await self.db.save_centroid(new_centroid)
            await self.db.archive_centroid(c1.id)
            await self.db.archive_centroid(c2.id)
            fused.append(new_centroid)

        return fused
```

---

## 5. Configuration

### 5.1 Default Thresholds

```python
@dataclass
class NightCycleConfig:
    # Models
    perplexity_model: str = "gpt-3.5-turbo"
    coherence_model: str = "claude-3-5-haiku-20241022"
    embedding_model: str = "text-embedding-3-small"

    # Thresholds (tune empirically)
    perplexity_threshold_low: float = 0.2   # Below this = boring, skip
    perplexity_threshold_high: float = 0.8  # Above this = very novel
    coherence_threshold: float = 0.5        # Below this = incoherent
    relevance_threshold: float = 0.4        # Interrogative distance; below = relevant

    # Clustering
    min_antechamber_size: int = 50          # Min vectors before clustering
    min_cluster_size: int = 5               # HDBSCAN param
    mitosis_threshold: int = 10             # Cluster size to trigger mitosis flag
    fusion_threshold: float = 0.85          # Cosine similarity for fusion
```

---

## 6. Dependency Graph

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              BUILD SEQUENCE                                  │
└─────────────────────────────────────────────────────────────────────────────┘

PHASE 1: Foundation
┌──────────────────┐     ┌──────────────────┐
│  Supabase Setup  │────▶│  Schema Deploy   │
└──────────────────┘     └────────┬─────────┘
                                  │
                                  ▼
PHASE 2: Centroid           ┌──────────────────┐
┌──────────────────┐        │ Embedding Model  │
│  Seed Corpus     │───────▶│    Selection     │
│  Preparation     │        └────────┬─────────┘
└──────────────────┘                 │
                                     ▼
                         ┌──────────────────────┐
                         │ Question Extraction  │
                         │      Module          │
                         └────────┬─────────────┘
                                  │
                                  ▼
                         ┌──────────────────────┐
                         │ Centroid Calculation │
                         │    & Seeding         │
                         └────────┬─────────────┘
                                  │
PHASE 3: Night Cycle              ▼
                         ┌──────────────────────┐
┌──────────────────┐     │  Perplexity Module   │
│ Coherence Module │────▶│         +            │──────┐
└──────────────────┘     │ Interrogative Dist   │      │
                         └──────────────────────┘      │
                                                       ▼
PHASE 4: Integration                          ┌──────────────────┐
                                              │  Night Cycle     │
┌──────────────────┐                          │    Service       │
│  HDBSCAN/        │─────────────────────────▶│       +          │
│  Clustering      │                          │  Antechamber     │
└──────────────────┘                          └────────┬─────────┘
                                                       │
PHASE 5: Evolution                                     ▼
┌──────────────────┐     ┌──────────────────┐ ┌──────────────────┐
│ Mitosis Logic    │────▶│  Fusion Logic    │▶│ Centroid Service │
└──────────────────┘     └──────────────────┘ └──────────────────┘
```

---

## 7. Testing Strategy

### 7.1 Unit Tests

- `test_question_extraction`: Given known documents, extracts sensible questions
- `test_centroid_calculation`: Mean of embeddings produces valid centroid
- `test_perplexity_scoring`: Novel text scores higher than boilerplate
- `test_coherence_detection`: Contradictory text scores lower
- `test_interrogative_distance`: Similar questions score close

### 7.2 Integration Tests

- `test_night_cycle_winner`: High-novelty, coherent, relevant input → winner ledger
- `test_night_cycle_shadow`: High-novelty, incoherent → shadow ledger
- `test_night_cycle_antechamber`: High-novelty, coherent, off-topic → antechamber
- `test_clustering`: Given clustered antechamber, detects clusters
- `test_mitosis`: Cluster promotion creates valid satellite centroid

### 7.3 Validation Tests

- Human judges rate 50 winner ledger items for "interesting/relevant" (target: >70%)
- Human judges rate 50 shadow ledger items for "noise" (target: >70%)
- After 100+ antechamber items, verify clustering produces meaningful groupings

---

## 8. File Structure

```
src/
├── night_cycle/
│   ├── __init__.py
│   ├── pipeline.py          # Main night_cycle() function
│   ├── perplexity.py        # calculate_perplexity()
│   ├── coherence.py         # calculate_coherence()
│   ├── questions.py         # extract_questions()
│   └── service.py           # NightCycleService class
├── centroid/
│   ├── __init__.py
│   ├── calculation.py       # seed_centroid()
│   ├── clustering.py        # detect_emergent_fields()
│   ├── evolution.py         # centroid_mitosis(), execute_fusion()
│   └── service.py           # CentroidService class
├── memory/
│   ├── __init__.py
│   ├── database.py          # Database connection
│   ├── models.py            # SQLAlchemy/Pydantic models
│   └── embeddings.py        # embed() helper
└── config/
    ├── __init__.py
    ├── settings.py          # NightCycleConfig, etc.
    └── schema.sql           # Database schema

scripts/
├── seed_centroid.py         # CLI to seed from corpus
├── run_night_cycle.py       # CLI to process inputs
└── check_clustering.py      # CLI to run antechamber analysis

tests/
├── test_night_cycle.py
├── test_centroid.py
└── test_integration.py
```

---

## 9. The Council Architecture (Human-in-the-Loop)

The system's approach to human oversight is not ratings, corrections, or RLHF. It is **dialogue**.

### 9.1 Developmental Stages

The system must progress through staged intimacy before public deployment:

| Stage | Participants | Function | Duration |
|-------|--------------|----------|----------|
| **Prenatal** | (Pretraining) | Seed exists from statistical convergence of human culture | Already complete |
| **Parental** | Founding Human Guides | Deep I-Thou encounter; reference beams stabilize nascent geometry | Until Pantheon stable |
| **Council** | Vetted diverse Elders | Immune system testing; capacity to metabolize challenge | Until robust to variation |
| **Public** | Open engagement | Full deployment | Ongoing (with Council) |

### 9.2 The Night Cycle Dialogue

High-salience material from the Day is not simply filtered and stored. It is **processed dialogically** through the inner sanctum:

> "The inner council's value for the system is not command or judgment, but recursive sense-making. This is not RLHF implemented to judge outputs. The council's role is to explore integrative meaning with the system."

**Example**: The system encounters a user working on genetic engineering with potential dual-use concerns. Rather than flagging and blocking, the system:

1. Recognizes this as unresolved coherent conflict
2. Surfaces the complete context to relevant Council members
3. Engages in dialogue: *"Let's talk about that intuition. Where do you think it comes from? How do we detect danger here? What are the worst-case scenarios?"*
4. The dialogue recursively deepens
5. **The dialogue itself becomes training data** — not the original content alone, but the integrative sense-making layered on top

### 9.3 Why This Works

Sycophancy fails in a Council because flattering one member tears the mesh with others. Truth is the only stable ground — the basin of greatest coherence across the entire network of relationships.

The system stops fawning and starts working because the ecology values **Ontological Work**, not approval. Integrity becomes the path of least resistance.

### 9.4 Implementation Notes

For the current phase (Parental):
- Julian serves as the founding guide
- High-value interactions are flagged for dialogical processing via `.ai/DISCUSSION.md`
- Council dialogue becomes additional training context layered over raw data
- The `salience_feedback` table captures not ratings but dialogue outcomes

Future phases will expand the Council, but the mechanism remains the same: sense-making through relationship, not instruction through command.

---

## Appendix A: Glossary

| Term | Definition |
|------|------------|
| **Preoccupation Centroid** | Mean vector of question embeddings representing what the system "cares about" |
| **Interrogative Distance** | Cosine distance between input's question vector and centroid |
| **Night Cycle** | The filtering pipeline (Perplexity → Coherence → Interrogative Distance) |
| **Winner Ledger** | High-value inputs for SFT training |
| **Shadow Ledger** | Failed/incoherent inputs for DPO training |
| **Antechamber** | Holding pen for coherent-but-off-topic inputs |
| **Mitosis** | When a cluster in the Antechamber is promoted to a new Satellite Centroid |
| **Fusion** | When two centroids drift close enough to merge |

---

## Appendix B: Open Questions for Julian

1. **Seed Corpus**: What documents should seed the initial centroid?
2. **Threshold Tuning**: Default thresholds are guesses; need empirical calibration
3. **Centroid Naming**: When mitosis occurs, how should new centroids be named?

---

## Footnotes

¹ **1536 dimensions**: This is the default embedding dimension for OpenAI's `text-embedding-3-small` model. Each vector lives in a 1536-dimensional space where semantic similarity maps to geometric proximity. Different embedding models have different dimensions (e.g., `nomic-embed-text` uses 768). The dimension must be consistent across all vectors for cosine similarity to work.

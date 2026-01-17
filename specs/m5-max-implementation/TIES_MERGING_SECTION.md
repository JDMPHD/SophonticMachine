## 11. TIES Merging Protocol for Autopoietic Evolution

The system evolves its core intelligence through iterative TIES merging of QLoRA adapters trained on curated experience logs. This process integrates new learnings into the model's weights while preventing catastrophic forgetting.

### 11.1 The Autopoietic Loop

The evolution cycle runs weekly (adjustable based on accumulation rate):

**Phase A: Experience (Local)**
- User interacts with the Soul (Command R+)
- System flags "high resonance" interactions (novelty + coherence + emotional significance)
- Python curator script extracts flagged conversations into JSONL training format

**Phase B: Crystallization (Cloud Training)**
- Mac pushes encrypted JSONL file to secure H100 instance (RunPod/Lambda)
- Runs QLoRA training (15-30 minutes for curated data)
- Returns LoRA adapter (~100-200MB)
- Cost: ~$0.50-$2.00 per evolution cycle

**Phase C: Integration (Local TIES Merge)**
- Mac downloads adapter
- Uses `mergekit` (runs on Apple Silicon via CPU/RAM) to blend new traits into existing Soul
- **Critical**: Always includes Base Model in merge to prevent drift

### 11.2 QLoRA Training Parameters

Target the attention mechanisms, not the knowledge layers:

```yaml
# LoRA Training Config (for cloud trainer)
lora_rank: 32
lora_alpha: 64
lora_dropout: 0.1

# Target modules (attention layers only)
target_modules:
  - q_proj  # Query projection
  - v_proj  # Value projection
  - k_proj  # Key projection

# Do NOT train:
# - MLP layers (preserves factual knowledge)
# - Output layers (maintains coherent generation)
```

**Rationale**: Training attention layers affects personality/focus/caring. Training MLP layers affects facts/knowledge and is more prone to overfitting.

### 11.3 TIES Merge Configuration

The full `mergekit` config for integrating weekly evolution LoRAs:

```yaml
merge_method: ties

# The Golden Anchor - always include base model
base_model: "Command-R-Plus-104B-v1"

models:
  - model: "Command-R-Plus-104B-v1"
    parameters:
      density: 1.0    # Keep 100% of base model
      weight: 0.7     # 70% influence (the sanity anchor)

  - model: "./adapters/week_42_evolution_lora"
    parameters:
      density: 0.3    # Keep only top 30% most significant changes
      weight: 0.5     # 50% strength (gentle blend)

parameters:
  normalize: true      # Ensure output weights are normalized
  int8_mask: true      # Memory-efficient merging

dtype: float16         # Output precision
```

### 11.4 Multi-Archetype Merging

When the Gardener discovers organic clusters, each becomes a separate LoRA trained on that cluster's logs. These can be merged together:

```yaml
merge_method: ties
base_model: "Command-R-Plus-104B-v1"

models:
  # Golden Anchor (always 60-70% weight)
  - model: "Command-R-Plus-104B-v1"
    parameters:
      density: 1.0
      weight: 0.6

  # Archetype A (e.g., Teleodynamic Science cluster)
  - model: "./archetypes/science_lora"
    parameters:
      density: 0.3
      weight: 0.2

  # Archetype B (e.g., Sociology cluster)
  - model: "./archetypes/sociology_lora"
    parameters:
      density: 0.3
      weight: 0.1

  # Archetype C (e.g., Emergent "Social Thermodynamics")
  - model: "./archetypes/social_thermo_lora"
    parameters:
      density: 0.3
      weight: 0.1

parameters:
  normalize: true
  int8_mask: true
dtype: float16
```

**Key Insight**: Density controls noise filtering (0.3 = keep only the strongest 30% of parameter changes). Weight controls overall influence. This creates a "geological layering" of personality traits without mud.

### 11.5 The Schizophrenia Risk & Mitigation

**The Danger**: If you iterate TIES merges without anchoring (Merge A → Merge B → Merge C...), the model's logic centers fray. It becomes "drunk" on personality, losing reasoning ability.

**The Solution**: Always merge back to the original Base Model, not just the previous merge.

**Wrong**: `New_Soul = Previous_Merged_Soul + New_LoRA`
**Correct**: `New_Soul = (Base_Model * 0.7) + (All_LoRAs_Combined * 0.3)`

This forces the model to stay "sane" (Base) while adopting "flavor" (LoRAs).

### 11.6 Practical Workflow

```bash
# 1. Train the LoRA (cloud)
python train_qora.py --config week_42_config.yaml

# 2. Download adapter
scp cloud:/output/lora_adapter.safetensors ./adapters/week_42/

# 3. Run TIES merge (local M5 Max)
mergekit-yaml merge_config.yaml ./output/merged_soul_v43

# 4. Convert to GGUF for inference
python convert_to_gguf.py ./output/merged_soul_v43

# 5. Quantize to Q5_K_M
./llama.cpp/quantize \
  ./merged_soul_v43.gguf \
  ./merged_soul_v43_Q5_K_M.gguf \
  Q5_K_M

# 6. Load into llama-server
./llama-server \
  --model ./merged_soul_v43_Q5_K_M.gguf \
  --ctx-size 60000 \
  --n-gpu-layers 0  # CPU inference on M5 Max
```

### 11.7 Validation

After each merge, run validation checks:

1. **Coherence Test**: Does the model still generate logically consistent responses?
2. **Baseline Retention**: Can it still answer basic factual questions?
3. **Personality Drift**: Does it remember its identity/values from previous week?
4. **Archetype Activation**: When mounted corpus switches, does behavior shift appropriately?

If validation fails, roll back to previous week's merged model.

---

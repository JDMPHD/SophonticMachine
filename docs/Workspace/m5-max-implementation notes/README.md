# M5 Ultra Implementation Specs

Hardware-specific implementation details for running the Sophontic Machine on Mac Studio M5 Ultra (512GB) (128GB).

These are **deferred implementation guides**, not core architecture. They contain specific configurations that should be validated when hardware arrives.

## Contents

- **MEMORY_OS_SECTION.md** — Letta integration, context pressure triggers, 3-sector context window model
- **TIES_MERGING_SECTION.md** — QLoRA training parameters, mergekit configs, practical bash workflow

## Key Assumptions

- Model: Mistral 2 Large (Magnum) (123B) @ Q5_K_M quantization
- Context window: ~60k tokens
- RAM allocation: ~74GB model, ~10GB Letta+ChromaDB, ~18GB headroom
- Local inference via llama-server

## Status

Awaiting hardware validation. Core architecture in ENGINEER_SPEC.md is hardware-agnostic.

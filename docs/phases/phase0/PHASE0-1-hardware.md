# Phase 0-1 — Hardware & Infrastructure Requirements

## Status: In Progress

---

## Hardware Inventory

Specific hardware details (GPU model, RAM, CPU, storage) are stored in the private file:

**[`docs/private/hardware.md`](../../private/hardware.md)** ← gitignored, not public

That file contains the full spec tables for all hardware components. Reference it for exact values; do not copy specs back into this file.

### Summary for planning purposes (non-identifying)

- Discrete GPU: mid-range mobile workstation GPU with **4 GB VRAM** — this is the binding constraint
- Integrated GPU: excluded from all inference decisions
- System RAM: sufficient for CPU offloading layers that exceed VRAM (see private/hardware.md)
- CPU: capable of multi-threaded offload via llama.cpp (see private/hardware.md)
- Storage: internal drive sufficient; additional TB-scale external drives available for model weight storage (see private/hardware.md)

---

## VRAM Constraint Analysis

The discrete GPU has **4 GB VRAM**. This is the single most important hardware constraint for this project.

### What fits at 4 GB VRAM

| Model Size | Quantization | Approx VRAM | Fits? |
|---|---|---|---|
| 2–3B params | fp16 | ~4–6 GB | Marginal / No |
| 2–3B params | int8 | ~2–3 GB | Yes |
| 2–3B params | int4 (Q4_K_M) | ~1.5 GB | Yes |
| 7B params | fp16 | ~14 GB | No |
| 7B params | int4 (Q4_K_M) | ~4 GB | Marginal |
| 13B params | int4 | ~7 GB | No (GPU only) |
| 13B+ params | int4 + CPU offload | Varies | Possible via llama.cpp |

### External Drive Storage for Model Weights

TB-scale external drives are available and viable for storing model weights. Interface speed affects startup/load time; whether the drive must stay connected during inference depends on the serving backend chosen in [PHASE0-2-serving.md](PHASE0-2-serving.md).

| Interface | Typical read speed | Load time for ~20 GB model |
|---|---|---|
| USB 3.0 HDD | ~100 MB/s | ~3 min |
| USB 3.0 SSD | ~400 MB/s | ~50 sec |
| USB 3.1 Gen 2 SSD | ~800 MB/s | ~25 sec |
| Thunderbolt / NVMe | ~2000+ MB/s | ~10 sec |

External drives are well-suited for storing multiple model variants (different quants, different models) and swapping between them. Confirm interface type in private/hardware.md.

> **Note (llama.cpp):** If llama.cpp is chosen as the serving backend, it uses memory-mapped files (`mmap`) by default — the drive must stay connected throughout inference, and interface speed also affects per-request latency on first context load.

### Model Selection

**Binding constraint:** 4 GB VRAM + 16 GB RAM = ~20 GB total addressable memory.
**GPU compute capability:** 7.5 — meets the minimum requirement (7.0) for all evaluated models.

A model's weights must fit within addressable memory (VRAM + RAM). At Q4_K_M quantization (~0.5 bytes/param), the 20 GB limit accommodates models up to roughly 35B parameters with CPU offloading.

See **[AIMODEL.md](AIMODEL.md)** for the full model evaluation, feasibility analysis, and candidate shortlist.

---

## Checklist

- [x] Discrete GPU VRAM confirmed: 4 GB (see private/hardware.md)
- [x] Integrated GPU identified and excluded from inference
- [x] System RAM confirmed (see private/hardware.md)
- [x] Available disk space confirmed (see private/hardware.md)
- [x] CPU model confirmed (see private/hardware.md)
- [ ] **BLOCKER:** Base model selected (see AIMODEL.md for candidates)

---

## Key Decisions (fill in after checklist is complete)

| Decision | Choice | Notes |
|---|---|---|
| Base model | TBD — **BLOCKER** | Must be selected before all other decisions; see AIMODEL.md |
| Primary inference device | TBD | Depends on model and hardware decision above |
| Quantization level | TBD | Depends on model decision above |
| CPU offloading | TBD | Depends on model decision above |
| Run mode | TBD | On-demand vs. persistent background service |

---

## Next

Once RAM, disk, CPU, and model size are confirmed → proceed to [PHASE0-2-serving.md](PHASE0-2-serving.md) to select the serving backend.

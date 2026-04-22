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
- System RAM, CPU, and storage: see private file (TBD items remain)

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

### MiniMax-M2.7 VRAM Requirement

> **Action required:** Check the MiniMax-M2.7 model card for exact parameter count and recommended VRAM.

- If ≤ 7B parameters: fits on GPU with int4 quantization (GGUF Q4_K_M)
- If > 7B parameters: GPU-only inference not viable; use **llama.cpp with CPU offloading**
- If > ~40B parameters: expect slow inference (~1–3 tok/s) even with offloading; consider whether a smaller model is acceptable

---

## Checklist

- [x] Discrete GPU VRAM confirmed: 4 GB (see private/hardware.md)
- [x] Integrated GPU identified and excluded from inference
- [ ] System RAM confirmed (see private/hardware.md)
- [ ] Available disk space confirmed (see private/hardware.md)
- [ ] CPU model confirmed (see private/hardware.md)
- [ ] MiniMax-M2.7 parameter count and VRAM requirement looked up
- [ ] Decision made: GPU-only vs. CPU offload vs. CPU-only inference

---

## Key Decisions (fill in after checklist is complete)

| Decision | Choice | Notes |
|---|---|---|
| Primary inference device | TBD | GPU-only if model ≤ ~7B int4; offloaded otherwise |
| Quantization level | TBD | Q4_K_M recommended as starting point |
| CPU offloading | TBD | Depends on model size vs. VRAM |
| Run mode | TBD | On-demand vs. persistent background service |

---

## Next

Once RAM, disk, CPU, and model size are confirmed → proceed to [PHASE0-2-serving.md](PHASE0-2-serving.md) to select the serving backend.

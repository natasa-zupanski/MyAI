# AI Model Candidates

Research into open-weights models that fit within hardware constraints. Used to resolve the base model blocker identified in [PHASE0-1-hardware.md](PHASE0-1-hardware.md).

**Sources:** [Artificial Analysis Leaderboard](https://artificialanalysis.ai/leaderboards/models), individual HuggingFace model cards.

---

## Hardware Constraints (summary)

| Resource | Available | Notes |
|---|---|---|
| GPU VRAM | 4 GB | Binding limit for GPU-only inference |
| System RAM | 16 GB | Available for CPU offloading |
| Total addressable | ~20 GB | VRAM + RAM combined |
| GPU compute capability | 7.5 | Meets most models' minimum (7.0+) |

**Memory rule of thumb (Q4_K_M quantization):** ~0.5 bytes per parameter.
A 10B model ≈ 5 GB; a 27B model ≈ 14 GB; a 35B model ≈ 17.5 GB.

For more detail on hardware constraints, see [PHASE0-1-hardware.md](PHASE0-1-hardware.md).

---

## Model Overview

| Model | Params | Architecture | Q4_K_M size | Fits where | Intelligence score | License |
|---|---|---|---|---|---|---|
| NVIDIA Nemotron-3-Nano-4B | 3.97B | Mamba-2/Transformer hybrid | ~2 GB | VRAM (comfortable) | — (edge-focused) | NVIDIA Nemotron Open |
| Qwen3.5 4B | 4B | Dense | ~2 GB | VRAM (comfortable) | 27/100 | Apache 2.0 |
| **Qwen2.5-Coder-7B-Instruct** | **7.61B** | **Dense** | **~4.5 GB** | **Minor CPU offload** | **— (coding-specialized)** | **Apache 2.0** |
| Ministral 8B | 8B | Dense | ~4 GB | VRAM (full) | 15/100 | MistralAI Research |
| Qwen3.5 9B | 9.7B | Dense | ~5 GB | Minor CPU offload | 32/100 | Apache 2.0 |
| Qwen3.5 27B | 27.8B | Dense | ~14 GB | CPU offload | 42/100 | Apache 2.0 |
| Gemma 4 31B | 31B | Dense | ~15.5 GB | CPU offload (tight) | 32–39/100 | Gemma ToS |
| Qwen3.5 35B A3B | 35B / 3B active | MoE | ~17.5 GB | CPU offload (tight) | 31–37/100 | Apache 2.0 |

> **MoE note (A2B / A3B):** Only N billion parameters are active per token, but all parameters must still be loaded into memory. Memory usage scales with total params; inference speed scales closer to active params.

---

## Model Profiles

### NVIDIA Nemotron-3-Nano-4B ✅ Fits in VRAM
- **HuggingFace:** https://huggingface.co/nvidia/NVIDIA-Nemotron-3-Nano-4B-BF16
- **Parameters:** 3.97B
- **Architecture:** Mamba-2/Transformer hybrid (primarily Mamba-2 and MLP with only 4 attention layers) — compressed from Nemotron-Nano-9B-v2
- **Context window:** 262k tokens
- **Q4_K_M size:** ~2 GB → fits in VRAM with 2 GB to spare
- **License:** NVIDIA Nemotron Open Model License (commercial use permitted)
- **Quantizations available:** 35 variants (llama.cpp, LM Studio, Ollama, Jan)
- **Supported backends:** HuggingFace Transformers, vLLM, llama.cpp, TRT-LLM, SGLang
- **Designed for:** Edge and local deployment (Jetson, GeForce RTX, IoT)
- **Reasoning:** Controllable — enable/disable reasoning traces via `enable_thinking` parameter
- **Coding:** Explicitly listed as a primary use case; training data includes GitHub and synthetic code data

**Benchmark highlights:**

| Benchmark | Reasoning Off | Reasoning On |
|---|---|---|
| MATH500 | — | 95.4 |
| AIME25 | — | 78.5 |
| IFEval-Instruction | 88.0 | 92.0 |
| RULER 128k (long context) | 91.1 | — |
| GPQA | — | 53.2 |

**Notes:**
- Purpose-built for edge/local deployment — the closest architectural match to this project's constraints
- Mamba-2 hybrid is more memory-efficient than pure transformer for long contexts
- NVIDIA Nemotron license is not as permissive as Apache 2.0; review before commercial redistribution
- Tool calling supported natively

---

### Qwen2.5-Coder-7B-Instruct ✅ Minor CPU offload — **Selected starting model**
- **HuggingFace:** https://huggingface.co/Qwen/Qwen2.5-Coder-7B-Instruct
- **Parameters:** 7.61B dense
- **Context window:** 128k tokens
- **Q4_K_M size:** ~4.5 GB → ~0.5 GB spills to RAM via CPU offload
- **License:** Apache 2.0
- **Reasoning:** No dedicated reasoning mode; strong chain-of-thought on code tasks
- **Coding:** Purpose-built coding specialist — trained on code, math, and general text with emphasis on code generation, debugging, and explanation

**Benchmark highlights (coding-focused):**

| Benchmark | Score | Notes |
|---|---|---|
| HumanEval | ~88 | Code generation |
| MBPP | ~79 | Python programming problems |
| LiveCodeBench | Competitive | Real-world coding evaluation |
| MultiPL-E | Strong | Multi-language code generation |

**Notes:**
- Coding-specialized variant of Qwen2.5; outperforms general Qwen3.5 9B on coding tasks at a smaller memory footprint
- Well-suited as the default model for a local coding agent
- Dynamic model switching planned — can swap to Qwen3.5 27B for complex reasoning tasks; see serving backend selection in [PHASE0-2-serving.md](PHASE0-2-serving.md)
- Quantizations available via llama.cpp / Ollama / LM Studio community builds
- Apache 2.0 — no redistribution restrictions

---

### Qwen3.5 9B ✅ Minor CPU offload
- **Parameters:** 9.7B dense
- **Context window:** 262k tokens
- **Q4_K_M size:** ~5 GB → ~1 GB spills to RAM via CPU offload
- **License:** Apache 2.0
- **Reasoning:** Yes (extended thinking)
- **Intelligence score:** 32/100 (Artificial Analysis, ranked #6 among comparable models)
- **Quantizations:** Available via Ollama / llama.cpp community builds

**Notes:**
- Strong balance of quality and hardware fit
- Apache 2.0 is the most permissive license of all candidates
- Noticeably slower than smaller models (42 t/s reported via API) — expect lower on this hardware
- Qwen2.5-Coder-7B (coding-specialized variant from same family) may outperform this on coding tasks — **investigate separately**

---

### Qwen3.5 27B ✅ CPU offload
- **Parameters:** 27.8B dense
- **Context window:** 262k tokens
- **Q4_K_M size:** ~14 GB → runs primarily in RAM, partial GPU offload
- **License:** Apache 2.0
- **Reasoning:** Yes (extended thinking)
- **Intelligence score:** 42/100 (ranked #2 among 118 comparable models on Artificial Analysis)

**Notes:**
- Highest quality model that fits within the 20 GB constraint with headroom
- Inference will be slower than smaller models due to heavy CPU offloading
- Extended 262k context is well-suited for multi-file coding tasks
- Best quality-per-machine for this hardware if inference speed is acceptable

---

### Qwen3.5 35B A3B ⚠️ Tight fit
- **Parameters:** 35B total / ~3B active (MoE)
- **Context window:** 262k tokens
- **Q4_K_M size:** ~17.5 GB → leaves ~2.5 GB headroom against 20 GB limit
- **License:** Apache 2.0
- **Intelligence score:** 31–37/100

**Notes:**
- MoE means inference speed is closer to a 3B model despite 35B total weights — better throughput per GB than dense 27B
- Memory is very tight; KV cache and OS overhead could push it over the limit
- Worth testing but may require Q3_K_M or lower quantization to be stable

---

### Gemma 4 31B ⚠️ Tight fit
- **Parameters:** 31B dense
- **Context window:** 256k tokens
- **Q4_K_M size:** ~15.5 GB
- **License:** Gemma Terms of Service (more restrictive than Apache 2.0; prohibits certain uses)
- **Intelligence score:** 32–39/100

**Notes:**
- Quality comparable to Qwen3.5 27B
- License is more restrictive than Apache 2.0 alternatives
- No strong reason to prefer over Qwen3.5 27B given similar size and more permissive alternatives

---

## To Investigate

| Model | Reason | Status |
|---|---|---|
| ~~Qwen2.5-Coder-7B~~ | ~~Coding-specialized; outperforms general Qwen3.5 9B on coding tasks~~ | **Selected — see profile above** |
| Qwen2.5-Coder-14B | Larger coding-specialist; ~8 GB Q4_K_M — needs CPU offload but may offer quality lift | Not yet evaluated |
| Qwen2.5-Coder-32B | Largest coding-specialist; ~16 GB Q4_K_M — heavy CPU offload | Not yet evaluated |
| DeepSeek-Coder-V2-Lite (16B A2.4B MoE) | Coding-focused MoE; active params keep inference fast | Not yet evaluated |

---

## Summary Recommendation

| Priority | Model | Reason | Status |
|---|---|---|---|
| **Default / starting model** | **Qwen2.5-Coder-7B-Instruct** | **Coding-specialist; fits with minor CPU offload; Apache 2.0** | **Selected** |
| Heavy reasoning / complex tasks | Qwen3.5 27B | Highest quality that fits; swap in dynamically | Available for dynamic use |
| Fallback (VRAM-only) | Nemotron-3-Nano-4B | Fits fully in VRAM; designed for edge hardware | Available if needed |
| Investigate | Qwen2.5-Coder-14B / 32B | Larger coding specialists; may offer quality lift with CPU offload | Not yet evaluated |

**Dynamic model switching** is a first-class requirement: the serving backend (see [PHASE0-2-serving.md](PHASE0-2-serving.md)) must support loading and unloading models at runtime. Qwen2.5-Coder-7B-Instruct is the default; other models from this table can be loaded on demand.

# Phase 0-2 — Model Serving Backend Requirements

## Status: Complete

---

## Serving Backend Options

### Option A — llama.cpp / llama-server
| Property | Notes |
|---|---|
| Format | GGUF (quantized, single file) |
| GPU support | CUDA, Metal, Vulkan, CPU |
| CPU offloading | Yes — split layers between GPU and RAM |
| API | OpenAI-compatible `/v1/chat/completions` (llama-server) |
| Windows support | Yes, prebuilt binaries available |
| Best for | **4 GB VRAM scenario**; handles partial GPU offloading transparently |

### Option B — vLLM
| Property | Notes |
|---|---|
| Format | HuggingFace (safetensors / fp16) |
| GPU support | CUDA only |
| CPU offloading | No — model must fully fit in VRAM |
| API | OpenAI-compatible |
| Windows support | Limited (WSL2 recommended) |
| Best for | High-throughput GPU servers with sufficient VRAM |

### Option C — Ollama
| Property | Notes |
|---|---|
| Format | GGUF (via Modelfile) or pulled from Ollama registry |
| GPU support | CUDA, Metal |
| CPU offloading | Yes |
| API | OpenAI-compatible |
| Windows support | Yes, native installer |
| Best for | Simplest setup; less control over quantization parameters |

### Option D — LM Studio
| Property | Notes |
|---|---|
| Format | GGUF |
| GPU support | CUDA, Metal |
| CPU offloading | Yes |
| API | OpenAI-compatible (local server mode) |
| Windows support | Yes, GUI application |
| Best for | Non-developer setup; limited for automation/scripting |

---

## Recommendation

Given the Quadro T1000 Max-Q (4 GB VRAM) and Qwen2.5-Coder-7B-Instruct (~4.5 GB Q4_K_M):

- **Selected: llama.cpp / llama-server** — prebuilt CUDA binary; layer split handles the ~0.5 GB VRAM overflow transparently; OpenAI-compatible API; scriptable for dynamic model switching
- Context window set to 16384 by default; application layer controls this at server start time
- Dynamic model switching is handled at the application layer: stop the server, start a new instance with a different `--model` path

vLLM is ruled out (no CPU offloading). Ollama was considered but llama.cpp was preferred for control over exact quantization and GPU layer configuration.

---

## Checklist

- [x] Chosen model GGUF availability confirmed: Qwen2.5-Coder-7B-Instruct — community GGUF builds available on HuggingFace (Bartowski; Q4_K_M available)
- [x] Serving backend selected: llama.cpp / llama-server
- [x] Prebuilt binary with CUDA support (GitHub releases — no build from source required)
- [x] GPU layers: starting estimate ~28/32 layers on GPU (~3.9 GB VRAM); tune empirically during Phase 1 benchmarking
- [x] Context window: 16384 default; configurable at runtime via `--ctx-size` (set by application layer — not hardcoded)
- [x] Server port: 8000 (change if conflict found)

---

## Key Decisions

| Decision | Choice | Notes |
|---|---|---|
| Serving backend | llama.cpp / llama-server | Maximum control over layer offloading; OpenAI-compatible API; headless and scriptable |
| Installation method | Prebuilt binary (GitHub releases) | CUDA-enabled build; no compile step required |
| Model format | GGUF Q4_K_M | Single file; handled natively by llama.cpp; Bartowski builds available |
| GPU layers | ~28/32 starting point | Tune empirically in Phase 1; target ~3.9 GB VRAM leaving headroom for KV cache |
| Context window | 16384 default; runtime-configurable | Application layer passes `--ctx-size` at server start; not hardcoded |
| Server port | 8000 | Change if conflict found |
| Dynamic model switching | Application-layer restart | llama-server serves one model at a time; switching requires stop/start with new `--model` path — application layer manages this |

---

## Next

After selecting serving backend → proceed to [PHASE0-3-api-interface.md](PHASE0-3-api-interface.md).

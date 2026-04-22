# Phase 0-2 — Model Serving Backend Requirements

## Status: Pending (blocked on PHASE0-1 hardware decisions)

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

## Recommendation (pending hardware confirmation)

Given the Quadro T1000 Max-Q (4 GB VRAM):

- **Primary recommendation: llama.cpp / llama-server** — maximum control over layer offloading, runs headless, scriptable, OpenAI-compatible, best option when model exceeds VRAM
- **Fallback: Ollama** — if ease of setup is prioritized and llama.cpp build proves difficult on Windows

vLLM is **not recommended** for this hardware unless the model is confirmed to fit entirely within 4 GB.

---

## Checklist

- [ ] MiniMax-M2.7 GGUF availability confirmed (check HuggingFace for community GGUF conversion or official release)
- [ ] Serving backend selected based on model size vs. VRAM
- [ ] Decide: build llama.cpp from source (CUDA support) or use prebuilt binary
- [ ] Decide: number of GPU layers to offload (`--n-gpu-layers` in llama.cpp)
- [ ] Decide: context window size (`--ctx-size`; larger = more RAM/VRAM needed)
- [ ] Decide: target port for model server (default: `8000`)

---

## Key Decisions

| Decision | Choice | Notes |
|---|---|---|
| Serving backend | TBD | |
| Model format | TBD | GGUF preferred for llama.cpp/Ollama |
| GPU layers | TBD | Set after benchmarking VRAM usage |
| Context window | TBD | 4096 minimum for coding; 8192+ preferred |
| Server port | 8000 | Unless conflict found |

---

## Next

After selecting serving backend → proceed to [PHASE0-3-api-interface.md](PHASE0-3-api-interface.md).

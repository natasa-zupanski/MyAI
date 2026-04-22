# MyAI — Local AI Platform Plan

## Overview

Build a locally hosted AI platform using MiniMax-M2.7 as the base model, initially focused on coding assistance but architected to support additional applications (writing, research, document Q&A, automation, etc.) without rework.

---

## Phase 0 — Requirements Collection

**Goal:** Define constraints and decisions before writing any code.

### 0.1 Hardware & Infrastructure Requirements
- [ ] Document available hardware (CPU, RAM, GPU model, VRAM)
- [ ] Determine whether MiniMax-M2.7 fits in VRAM or requires CPU offloading / quantization
- [ ] Decide on quantization level (fp16, int8, int4) based on hardware limits
- [ ] Confirm storage capacity for model weights (MiniMax-M2.7 is ~15–20GB+ depending on quant)
- [ ] Decide whether the service needs to run 24/7 or on-demand

### 0.2 Model Serving Requirements
- [ ] Evaluate serving backends: **vLLM**, **llama.cpp / llama-server**, **Ollama**, **LM Studio**
  - vLLM: best throughput, GPU-only, OpenAI-compatible API
  - llama.cpp: CPU+GPU, widest format support (GGUF), lightweight
  - Ollama: easiest local setup, good model management
- [ ] Decide on target inference latency for interactive coding use
- [ ] Confirm whether multi-user / concurrent requests are needed now or later

### 0.3 API & Interface Requirements
- [ ] Adopt an OpenAI-compatible REST API (`/v1/chat/completions`, `/v1/completions`) so IDE plugins and future apps work without custom clients
- [ ] Decide on authentication strategy (none for localhost, token-based if exposing on LAN)
- [ ] Decide on a web UI for direct interaction (e.g., Open WebUI, custom React app)
- [ ] List IDE integrations needed (VS Code Continue extension, JetBrains AI Assistant, etc.)

### 0.4 Coding Application Requirements
- [ ] Define supported tasks: code completion, inline chat, code explanation, refactoring, test generation, docstring generation, debugging help
- [ ] List target languages/frameworks (Python, TypeScript, etc.)
- [ ] Decide whether a system prompt / instruction template is needed for coding focus
- [ ] Determine context window needs (long files, multi-file context)
- [ ] Decide on RAG over local codebase (vector store for project-aware suggestions)

### 0.5 Extensibility Requirements
- [ ] Define the application model: each future "app" is a module that provides a system prompt, tool definitions, and optional RAG pipeline
- [ ] Decide how apps are registered and switched (config file, API parameter, UI selector)
- [ ] List known future applications to design for (e.g., document Q&A, writing assistant, task automation)
- [ ] Decide on a plugin/tool-use strategy (function calling, MCP server, custom tool router)

### 0.6 Data & Privacy Requirements
- [ ] Confirm all inference stays on-device (no telemetry to external services)
- [ ] Decide on conversation logging (local SQLite, plain files, or none)
- [ ] Decide on vector store for RAG (local: Chroma, Qdrant, FAISS)

---

## Phase 1 — Model & Serving Layer

**Goal:** Get MiniMax-M2.7 running locally with a stable, OpenAI-compatible API endpoint.

- [ ] Download MiniMax-M2.7 weights (HuggingFace or official release)
- [ ] Convert to serving format if needed (GGUF for llama.cpp, or use HF directly with vLLM)
- [ ] Configure chosen serving backend with appropriate quantization
- [ ] Expose `http://localhost:8000` OpenAI-compatible API
- [ ] Write a smoke test script that hits `/v1/chat/completions` and validates a response
- [ ] Document startup command and system requirements in README

---

## Phase 2 — Core Platform Layer

**Goal:** Build a thin orchestration layer above the model server that handles routing, app context injection, and future extensibility.

- [ ] Define `AppConfig` schema: `name`, `system_prompt`, `tools[]`, `rag_pipeline` (optional)
- [ ] Build a config loader that reads app definitions from `apps/` directory
- [ ] Build a request router that prepends the correct system prompt and tools for the active app
- [ ] Expose `/api/chat` endpoint (wraps model server, adds app context)
- [ ] Add conversation history management (in-memory, with optional persistence)
- [ ] Write integration tests for router and app injection

---

## Phase 3 — Coding Application

**Goal:** First-class coding assistant as the initial application.

- [ ] Write `apps/coding.yaml` with a focused system prompt (code-first, concise, no unnecessary prose)
- [ ] Integrate with VS Code via the **Continue** extension (points to local `/v1` endpoint)
- [ ] Optional: add RAG over local project files using a local vector store (Chroma + sentence-transformers)
- [ ] Test on real coding tasks: completion, refactor, explain, test generation
- [ ] Tune system prompt based on quality of responses

---

## Phase 4 — Web UI

**Goal:** Browser-based chat interface for direct interaction with any registered app.

- [ ] Evaluate UI options: **Open WebUI** (drop-in, feature-rich) vs custom React app
- [ ] Deploy chosen UI pointed at local API
- [ ] Add app selector in UI (switch between coding, writing, etc.)
- [ ] Ensure conversation history is visible and clearable

---

## Phase 5 — Additional Applications (Future)

Each new application is a new file in `apps/` plus any app-specific tooling.

- [ ] `apps/writing.yaml` — writing and editing assistant
- [ ] `apps/docs_qa.yaml` — Q&A over local documents (PDF, Markdown) via RAG
- [ ] `apps/shell.yaml` — shell command helper with tool-use for running commands
- [ ] `apps/research.yaml` — summarization and synthesis with web search tool

---

## Architecture Diagram (Target State)

```
┌──────────────────────────────────────────────────────┐
│                    Clients                           │
│  VS Code (Continue)  │  Web UI  │  Future Apps       │
└────────────┬─────────┴────┬─────┴────────────────────┘
             │              │
             ▼              ▼
┌────────────────────────────────────────────────────┐
│              Platform API  (:3000)                 │
│   /api/chat → app context injection → router       │
│   app registry  │  history store  │  tool router   │
└───────────────────────────┬────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────┐
│           Model Server  (:8000)                    │
│         MiniMax-M2.7  (vLLM / llama.cpp)           │
│         OpenAI-compatible REST API                 │
└────────────────────────────────────────────────────┘
```

---

## Key Design Decisions

| Decision | Choice | Rationale |
|---|---|---|
| API standard | OpenAI-compatible | IDE plugins and UIs work without custom clients |
| App isolation | YAML config per app | Simple to add new apps, no code changes |
| Model serving | TBD (Phase 0) | Depends on hardware (VRAM vs CPU) |
| Vector store | Chroma (local) | Fully local, easy Python integration |
| UI | Open WebUI (preferred) | Battle-tested, no frontend build needed |

---

## Next Action

**Complete Phase 0** — fill in the hardware and preference checklist above. Once those answers are known, Phase 1 choices (serving backend, quantization) can be locked in and implementation can begin.

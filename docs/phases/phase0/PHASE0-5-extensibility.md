# Phase 0-5 — Extensibility Architecture Requirements

## Status: Complete

---

## Goal

Design the platform so that adding a new application (writing assistant, document Q&A, shell helper, etc.) requires only:
1. A new YAML config file in `apps/`
2. Optionally, a new tool definition or RAG pipeline module

No changes to the core serving, routing, or API layers.

---

## Application Model

Each "app" is defined by a config file with the following schema:

```yaml
# apps/<name>.yaml
name: coding                          # unique identifier
display_name: Coding Assistant        # shown in UI
description: Code-first assistant     # shown in UI

model: qwen2.5-coder-7b-instruct      # model to load for this app; overrides platform default

system_prompt: |
  You are a coding assistant. Be concise...

tools: []                             # optional; omit or leave empty if app uses no tools

rag:                                  # optional; omit entirely if app uses no RAG
  enabled: false
  index_path: null                    # path to Chroma collection
  top_k: 5                            # number of chunks to inject

model_overrides:                      # optional per-app model params
  temperature: 0.2
  max_tokens: 2048
```

`tools` and `rag` are optional — omit them entirely from an app config if not needed. The platform treats their absence the same as empty/disabled. `model` enables per-app model selection and directly supports the dynamic model switching requirement from [PHASE0-1-hardware.md](PHASE0-1-hardware.md).

---

## App Registry

The platform loads all `apps/*.yaml` files on startup. Apps are addressable by `name` in API requests:

```json
POST /api/chat
{
  "app": "coding",
  "messages": [...]
}
```

If `app` is omitted, a default app is used (configured in `config.yaml`).

Default app is `coding`. Configured in `config.yaml` at the platform level.

---

## Tool / Function Calling Strategy

Qwen2.5-Coder-7B-Instruct supports function calling natively. Tools are defined per-app in the YAML and executed by the platform (server-side).

### Hybrid execution model

The platform owns the agentic tool loop:
1. Model emits a tool call
2. Platform executes the tool (e.g. `read_file`, `run_shell_command`)
3. Platform injects the result and continues the conversation

Clients with native tool support (VS Code Continue, Open WebUI) do not need to use this loop — they can inject tool results directly as context messages in the request. The platform API accepts pre-executed results as context messages; no special handling is required on the platform side for this case.

This means:
- **Custom web UI / agentic workflows:** use the platform-side loop
- **Continue / Open WebUI:** inject tool results as context; bypass the platform loop

### Built-in tools (Phase 3+)

| Tool | Used By | Notes |
|---|---|---|
| `read_file` | coding, docs_qa | Reads local file content into context |
| `list_directory` | coding, shell | Lists files in a directory |
| `web_search` | research | External search; requires isolation review |
| `run_shell_command` | shell | Sandboxed execution only |

---

## Known Future Applications (priority order)

| Priority | App | Description | Key New Requirement |
|---|---|---|---|
| 1 | `writing-*` | Writing and editing assistant — multiple genre configs (e.g. `writing-technical.yaml`, `writing-creative.yaml`) | Multiple system prompts; no code tools; one YAML per genre |
| 2 | `docs_qa` | Q&A over local documents (PDF, Markdown) | RAG pipeline over document corpus (Phase 3) |
| 3 | `shell` | Shell command helper | `run_shell_command` tool (sandboxed) |
| 4 | `research` | Summarization and synthesis from web or local sources | `web_search` tool; isolation review required |
| 5 | `analysis` | Data analysis and reasoning | TBD — requirements not yet defined |

**Note on `writing`:** Multiple genres are handled by multiple YAML configs sharing the same app type, not a single config with a genre parameter. This keeps the app model simple and consistent.

---

## Checklist

- [x] App config YAML schema finalized: `tools` and `rag` are optional (omit if unused); `model` field added for per-app model selection
- [x] App registry mechanism decided: load all `apps/*.yaml` on startup; addressable by `name` in API requests
- [x] Default app: `coding`
- [x] Tool calling strategy decided: platform-side execution; clients may inject pre-executed results as context instead
- [x] Future app priority order set: writing (multi-genre), docs_qa, shell, research, analysis

---

## Key Decisions

| Decision | Choice | Notes |
|---|---|---|
| App config format | YAML | Human-readable; one file per app; multiple configs per app type (e.g. writing genres) |
| App model field | Per-app `model:` in YAML | Overrides platform default; enables dynamic model switching per app |
| Tool execution | Platform-side (hybrid) | Platform executes tools in agentic loop; clients may inject results as context instead |
| Default app | `coding` | Configured in platform `config.yaml` |
| Future app build order | writing → docs_qa → shell → research → analysis | Writing first; multiple genre configs instead of a single writing app |

---

## Next

→ [PHASE0-6-data-privacy.md](PHASE0-6-data-privacy.md)

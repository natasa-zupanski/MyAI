# Phase 0-5 — Extensibility Architecture Requirements

## Status: Pending

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

system_prompt: |
  You are a coding assistant. Be concise...

tools: []                             # list of tool definitions (function calling)

rag:
  enabled: false
  index_path: null                    # path to Chroma collection
  top_k: 5                            # number of chunks to inject

model_overrides:                      # optional per-app model params
  temperature: 0.2
  max_tokens: 2048
```

- [ ] **TODO:** Review schema — confirm all fields needed at launch vs. later

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

- [ ] **TODO:** Decide default app name (suggestion: `coding`)

---

## Tool / Function Calling Strategy

The chosen model's function calling support will be confirmed once the base model is selected. Tools can be defined per-app in the YAML and forwarded to the model server.

Built-in tools to consider for future apps:
| Tool | Used By |
|---|---|
| `read_file` | coding, docs_qa |
| `list_directory` | coding, shell |
| `web_search` | research |
| `run_shell_command` | shell (sandboxed) |

- [ ] **TODO:** Confirm chosen model's function calling support and format from model card
- [ ] **TODO:** Decide whether tools are called by the model and executed by the platform, or passed through to the client

---

## Known Future Applications

| App | Description | Key New Requirement |
|---|---|---|
| `writing` | Writing and editing assistant | Different system prompt; no code tools |
| `docs_qa` | Q&A over local documents (PDF, Markdown) | RAG pipeline over document corpus |
| `shell` | Shell command helper | `run_shell_command` tool (sandboxed) |
| `research` | Summarization and synthesis | `web_search` tool |

- [ ] **TODO:** Prioritize which future app is built first after `coding`

---

## Checklist

- [ ] App config YAML schema finalized
- [ ] App registry mechanism decided (load from `apps/` on startup)
- [ ] Default app configured
- [ ] Tool calling strategy decided
- [ ] Future app priority order set

---

## Key Decisions

| Decision | Choice | Notes |
|---|---|---|
| App config format | YAML | Human-readable, easy to add new apps |
| Tool execution | TBD | Platform-side vs. client-side |
| Default app | TBD | Likely `coding` |
| First future app | TBD | |

---

## Next

→ [PHASE0-6-data-privacy.md](PHASE0-6-data-privacy.md)

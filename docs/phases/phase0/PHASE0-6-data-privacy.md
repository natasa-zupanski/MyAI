# Phase 0-6 — Data & Privacy Requirements

## Status: Pending

---

## Core Privacy Requirement

**All inference must stay on-device.** No conversation data, prompts, or code snippets are sent to any external service. This is a hard requirement that rules out any cloud model API as the backend.

---

## Telemetry & Phoning Home

| Component | Telemetry Risk | Mitigation |
|---|---|---|
| llama.cpp / llama-server | None (open source, no telemetry) | N/A |
| Ollama | Minimal usage stats | Can run in `OLLAMA_NOTRACK=1` mode |
| Open WebUI | None by default | Verify in release notes before deploying |
| VS Code Continue extension | Anonymized usage telemetry | Disable in `config.json`: `"telemetry": false` |

- [ ] **TODO:** Audit whichever serving backend and UI are chosen for telemetry and disable it

---

## Conversation Logging

Decide whether to persist chat history.

| Option | Pros | Cons |
|---|---|---|
| No persistence | Simplest, zero data at rest | History lost on restart |
| Local SQLite | Searchable history, lightweight | Sensitive code snippets at rest on disk |
| Plain JSON files | Human-readable | Harder to query |

- [ ] **TODO:** Decide on conversation persistence strategy
- [ ] If persisting: decide encryption at rest (Windows NTFS encryption or application-level)

---

## Vector Store (RAG)

If RAG is enabled (Phase 3+), embeddings and chunked source code are stored locally.

| Component | Storage |
|---|---|
| Chroma | Local directory (`./data/chroma/`) |
| Embedding model | Local (sentence-transformers, runs on CPU) |

No data leaves the machine.

---

## Model Weights

Model weights downloaded from HuggingFace are stored locally. HuggingFace download requires internet access once; after that the model runs fully offline.

- [ ] **TODO:** Decide model storage path (suggestion: `C:\Users\localmgr\.cache\models\` or project-local `models\` directory)
- [ ] **TODO:** Decide whether the model server should be reachable only on `127.0.0.1` (localhost-only) or `0.0.0.0` (LAN-accessible)

---

## Checklist

- [x] All inference on-device confirmed as hard requirement
- [ ] Telemetry audit plan in place for chosen components
- [ ] Conversation persistence decision made
- [ ] Model storage path decided
- [ ] Network binding decided (localhost vs. LAN)

---

## Key Decisions

| Decision | Choice | Notes |
|---|---|---|
| External API calls | None | Hard requirement |
| Conversation logging | TBD | |
| Model storage | TBD | |
| Network binding | TBD | localhost-only recommended |

---

## Next

Phase 0 complete → proceed to [PHASE1.md](../phase1/PHASE1.md) — Model & Serving Layer.

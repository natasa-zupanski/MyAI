# Phase 0-3 — API & Interface Requirements

## Status: Pending

---

## API Layer

### Standard: OpenAI-Compatible REST API

The platform will expose an OpenAI-compatible API. This is a non-negotiable design choice — it means:

- IDE plugins (Continue, Cursor, JetBrains AI) work without custom clients
- Web UIs (Open WebUI, etc.) connect out of the box
- Future applications can use the `openai` Python/JS SDK pointed at `localhost`
- No vendor lock-in to any specific tool

**Required endpoints (minimum viable):**
- `POST /v1/chat/completions` — main chat/instruction endpoint
- `GET /v1/models` — model list (required by some clients)

**Optional but useful:**
- `POST /v1/completions` — raw completion (some older IDE tools use this)
- `GET /v1/health` — health check for monitoring

### Platform API (wraps model server)

A thin platform layer sits in front of the model server to inject app context (system prompts, tools). It exposes:

- `POST /api/chat` — app-aware chat (adds system prompt and tool definitions for the selected app)
- `GET /api/apps` — list registered apps
- `GET /v1/*` — pass-through to model server for direct OpenAI-compatible access

---

## Authentication

| Scenario | Strategy |
|---|---|
| Localhost only | No auth (default) |
| LAN access | Static API key in `Authorization: Bearer <token>` header |
| Public exposure | Not recommended; out of scope for this plan |

- [ ] **TODO:** Decide whether LAN access is needed now or can be added later

---

## Web UI

### Option A — Open WebUI
- Full-featured chat interface, supports multiple models and conversations
- Docker or pip install, no frontend build needed
- Connects to OpenAI-compatible endpoints out of the box
- Supports system prompts, conversation history, model switching

### Option B — Custom React/Next.js app
- Full control over UX
- Requires frontend development time
- Better for app-specific UI (e.g., code diff view, file context panel)

**Recommendation:** Start with Open WebUI (zero build cost), switch to custom UI only if Open WebUI lacks a required feature.

- [ ] **TODO:** Confirm Open WebUI is acceptable as the initial UI

---

## IDE Integration

### VS Code — Continue Extension
- Open-source IDE extension for AI coding assistance
- Supports any OpenAI-compatible endpoint
- Provides: inline completions, sidebar chat, code actions (explain, refactor, test)
- Config: `.continue/config.json` points to `http://localhost:3000/v1`

### JetBrains — AI Assistant (optional)
- Supports custom OpenAI-compatible endpoints via plugin settings
- Lower priority; add if user uses JetBrains IDEs

- [ ] **TODO:** Confirm VS Code Continue is the primary IDE integration target

---

## Checklist

- [ ] OpenAI-compatible API confirmed as the standard
- [ ] LAN auth decision made
- [ ] Web UI selection made (Open WebUI vs. custom)
- [ ] IDE integration target confirmed (VS Code Continue)
- [ ] Platform API port decided (suggestion: `3000`, separate from model server `8000`)

---

## Key Decisions

| Decision | Choice | Notes |
|---|---|---|
| API standard | OpenAI-compatible | Locked |
| Authentication | TBD | None or static key |
| Web UI | TBD | Open WebUI preferred |
| IDE plugin | TBD | VS Code Continue preferred |
| Platform API port | 3000 | Model server on 8000 |

---

## Next

→ [PHASE0-4-coding-app.md](PHASE0-4-coding-app.md)

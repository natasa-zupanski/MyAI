# Phase 0-3 — API & Interface Requirements

## Status: Complete

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
| Localhost only | No auth (default) — **selected** |
| LAN access | Deferred — add static API key in `Authorization: Bearer <token>` header if needed later |
| Public exposure | Not recommended; out of scope for this plan |

LAN access is not required now. All services bind to `127.0.0.1` only.

---

## Web UI

### Selected: Open WebUI (initial) → Custom UI (future)

Open WebUI is used first to experiment and learn what features matter before building a custom UI. It is open source (MIT), self-hosted, and connects to the platform API via the OpenAI-compatible endpoint.

**Future:** A custom React/Next.js UI is planned once requirements are clear from Open WebUI usage. This is explicitly deferred — do not design for it in Phase 0.

### Open WebUI — Privacy Isolation Steps

Open WebUI has optional features that can make outbound network calls (update checks, community sharing, web search). The following steps ensure it runs in a fully isolated, local-only configuration.

#### Environment variables to set at startup

| Variable | Value | Purpose |
|---|---|---|
| `ENABLE_UPDATE_CHECK` | `false` | Disables version check calls to external servers |
| `ENABLE_COMMUNITY_SHARING` | `false` | Disables sharing features |
| `WEBUI_AUTH` | `false` | Disables login (localhost-only; re-enable if LAN access is added) |
| `WEBUI_URL` | `http://localhost:8080` | Binds UI base URL to localhost |

> **Note:** Verify these variable names against the Open WebUI release in use — names can change between versions. Source: https://docs.openwebui.com/getting-started/env-configuration

#### Binding to localhost only

- Run Open WebUI bound to `127.0.0.1`, not `0.0.0.0`
- If using Docker: expose port as `127.0.0.1:8080:8080` (not `8080:8080`)
- If using pip: pass `--host 127.0.0.1` to the server start command

#### Do not configure

- Web search integration (Searxng, Google, etc.)
- Image generation (DALL-E, Stable Diffusion external APIs)
- External model pulls from Ollama registry or HuggingFace within the UI

#### Optional — Windows Firewall rule (strongest isolation)

Add an outbound firewall rule blocking the Open WebUI process (or its Python/Docker process) from making external connections. This enforces isolation at the OS level regardless of application config.

#### Port

Open WebUI runs on port `8080` (default). Model server remains on `8000`; platform API on `3000`.

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

VS Code + Continue is confirmed as the primary IDE integration target.

---

## Checklist

- [x] OpenAI-compatible API confirmed as the standard
- [x] LAN auth decision made: localhost only, no auth; LAN deferred
- [x] Web UI selected: Open WebUI (isolated) as starting point; custom UI planned for future
- [x] Open WebUI isolation steps documented (env vars, localhost binding, no external integrations)
- [x] IDE integration target confirmed: VS Code + Continue extension
- [x] Platform API port: 3000 (model server: 8000, Open WebUI: 8080)

---

## Key Decisions

| Decision | Choice | Notes |
|---|---|---|
| API standard | OpenAI-compatible | Non-negotiable — enables IDE plugins, web UIs, and future apps without custom clients |
| Authentication | None (localhost only) | LAN access and static key auth deferred until needed |
| Web UI | Open WebUI (isolated) | MIT, self-hosted; run with isolation env vars and bound to 127.0.0.1; custom UI planned for future |
| IDE plugin | VS Code + Continue | Open-source; OpenAI-compatible endpoint; inline completions + sidebar chat |
| Platform API port | 3000 | Model server: 8000; Open WebUI: 8080 |
| Future Web UI | Custom React/Next.js | Deferred — use Open WebUI first to determine requirements |

---

## Next

→ [PHASE0-4-coding-app.md](PHASE0-4-coding-app.md)

# Phase 0-4 — Coding Application Requirements

## Status: Complete

---

## Supported Tasks

Define which coding tasks the assistant must handle at launch vs. nice-to-have later.

### Must Have (v1)
- [x] **Code completion** — suggest next lines or complete a function given context
- [x] **Inline chat** — ask questions about selected code in the editor
- [x] **Code explanation** — explain what a block of code does in plain language
- [x] **Refactoring suggestions** — improve readability, performance, or style
- [x] **Debugging help** — explain an error, suggest a fix given stack trace + code
- [x] **Docstring / comment generation** — generate function/class documentation

### Nice to Have (v1.5+)
- [ ] **Test generation** — write unit tests for a given function
- [ ] **Code translation** — convert code between languages
- [ ] **Multi-file context** — answer questions about code across multiple files
- [ ] **Shell command help** — generate or explain terminal commands

---

## Target Languages & Frameworks

Primary languages to prioritize in system prompt and context:

- Python
- TypeScript / JavaScript
- Java
- C#
- Go
- Rust

Supporting (included but lower priority):
- Bash / Shell
- SQL

Frameworks deferred — add to system prompt as specific projects require them.

---

## System Prompt Design

The coding app system prompt should:
- Establish the assistant as a code-first, concise helper
- Suppress unnecessary prose (no "Great question!" or lengthy preambles)
- Instruct the model to prefer idiomatic, readable code over clever one-liners
- Instruct the model to default to no comments unless logic is non-obvious
- Specify preferred languages if needed

Draft system prompt (refine in Phase 3):
```
You are a coding assistant. Before writing code, ask clarifying questions if the request is ambiguous,
then briefly state your plan before implementing. Write readable, clean code. Avoid clever one-liners
at the expense of clarity. Add comments only when the logic is non-obvious. Output only the relevant
code unless asked to explain. Default to the language of the surrounding context unless specified otherwise.
```

Key behaviors locked for v1:
- Ask clarifying questions before coding if the request is ambiguous
- State a short plan before implementing
- Readable and clean code; no style preferences beyond that
- No comments unless logic is non-obvious

Refine wording for Qwen2.5-Coder-7B-Instruct response style in Phase 3.

---

## Context Window & File Context

| Requirement | Value |
|---|---|
| Minimum context window | 4,096 tokens |
| Preferred context window | 8,192–32,768 tokens |
| Multi-file context | Not required at launch; add via RAG in Phase 3 |

Qwen2.5-Coder-7B-Instruct supports up to 128k tokens. Runtime context window is set to 16384 by default (configurable) — confirmed in [PHASE0-2-serving.md](PHASE0-2-serving.md).

---

## RAG Over Local Codebase (Optional, Phase 3)

For project-aware suggestions (e.g., "use the existing `UserService` pattern"), a RAG pipeline can index local files and inject relevant snippets into the context window.

- Vector store: **Chroma** (local, Python-native)
- Embedding model: **sentence-transformers** (runs locally, CPU)
- Trigger: per-project, index on first open or on file change

**Decision: deferred to Phase 3.** Performance without RAG needs to be explored first — the 16384-token context window covers single-file and manually pasted multi-file context at launch. Revisit once real usage patterns are clearer.

---

## Checklist

- [x] Supported tasks list confirmed (v1 must-haves locked; nice-to-haves deferred)
- [x] Target languages defined: Python, TS/JS, Java, C#, Go, Rust (primary); Bash, SQL (supporting)
- [x] System prompt drafted: clarify → plan → implement; readable clean code; no unnecessary comments
- [x] Context window confirmed: 128k max (model); 16384 default runtime (see PHASE0-2)
- [x] RAG deferred to Phase 3 — explore performance without it first

---

## Key Decisions

| Decision | Choice | Notes |
|---|---|---|
| Target languages | Python, TS/JS, Java, C#, Go, Rust | Bash and SQL as supporting; frameworks added per project |
| System prompt behavior | Clarify → plan → implement | Ask before coding if ambiguous; state plan before implementing |
| Code style | Readable and clean; no comments unless non-obvious | No further style preferences at this stage |
| System prompt wording | Draft locked; refine in Phase 3 | Tune wording for Qwen2.5-Coder-7B-Instruct response style |
| Context window | 16384 default (runtime-configurable) | Model supports 128k; runtime limit set in PHASE0-2 |
| RAG at launch | Deferred to Phase 3 | Explore performance without RAG first |

---

## Next

→ [PHASE0-5-extensibility.md](PHASE0-5-extensibility.md)

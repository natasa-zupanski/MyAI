# Phase 0-4 — Coding Application Requirements

## Status: Pending

---

## Supported Tasks

Define which coding tasks the assistant must handle at launch vs. nice-to-have later.

### Must Have (v1)
- [ ] **Code completion** — suggest next lines or complete a function given context
- [ ] **Inline chat** — ask questions about selected code in the editor
- [ ] **Code explanation** — explain what a block of code does in plain language
- [ ] **Refactoring suggestions** — improve readability, performance, or style
- [ ] **Debugging help** — explain an error, suggest a fix given stack trace + code
- [ ] **Docstring / comment generation** — generate function/class documentation

### Nice to Have (v1.5+)
- [ ] **Test generation** — write unit tests for a given function
- [ ] **Code translation** — convert code between languages
- [ ] **Multi-file context** — answer questions about code across multiple files
- [ ] **Shell command help** — generate or explain terminal commands

---

## Target Languages & Frameworks

- [ ] **TODO:** List the primary languages and frameworks used in your projects

Defaults to prioritize in system prompt:
- Python
- TypeScript / JavaScript
- Bash / Shell
- SQL

Add others as needed (Go, Rust, C#, etc.).

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
You are a coding assistant. Be concise. Output code directly without lengthy explanation unless asked.
Prefer idiomatic, readable solutions. When asked to fix code, output only the corrected code unless
asked to explain. Default to [PRIMARY_LANGUAGE] unless specified otherwise.
```

- [ ] **TODO:** Refine system prompt wording for MiniMax-M2.7 specifically (some models respond better to different prompt styles)

---

## Context Window & File Context

| Requirement | Value |
|---|---|
| Minimum context window | 4,096 tokens |
| Preferred context window | 8,192–32,768 tokens |
| Multi-file context | Not required at launch; add via RAG in Phase 3 |

- [ ] **TODO:** Confirm MiniMax-M2.7 maximum context window from model card

---

## RAG Over Local Codebase (Optional, Phase 3)

For project-aware suggestions (e.g., "use the existing `UserService` pattern"), a RAG pipeline can index local files and inject relevant snippets into the context window.

- Vector store: **Chroma** (local, Python-native)
- Embedding model: **sentence-transformers** (runs locally, CPU)
- Trigger: per-project, index on first open or on file change

- [ ] **TODO:** Decide whether RAG is needed at launch or deferred to Phase 3

---

## Checklist

- [ ] Supported tasks list confirmed
- [ ] Target languages defined
- [ ] System prompt drafted
- [ ] Context window requirement confirmed against model capability
- [ ] RAG decision made (launch vs. defer)

---

## Key Decisions

| Decision | Choice | Notes |
|---|---|---|
| Target languages | TBD | |
| System prompt | TBD | Draft above, finalize in Phase 3 |
| Context window | TBD | |
| RAG at launch | TBD | |

---

## Next

→ [PHASE0-5-extensibility.md](PHASE0-5-extensibility.md)

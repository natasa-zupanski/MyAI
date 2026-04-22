# MyAI

A locally hosted AI platform built on [MiniMax-M2.7](https://huggingface.co/MiniMaxAI). All inference runs on-device — no data leaves the machine.

## Purpose

The initial application is a coding assistant: code completion, inline chat, refactoring, debugging, and documentation generation, integrated into VS Code via the [Continue](https://www.continue.dev/) extension. The platform is designed to support additional applications (writing assistant, document Q&A, research, shell helper) without changes to the core serving or routing layers.

## Documentation

- [`docs/PLAN.md`](docs/PLAN.md) — full phased implementation plan
- [`docs/PHASE-DOC-GUIDE.md`](docs/PHASE-DOC-GUIDE.md) — conventions for phase documentation
- [`docs/phases/`](docs/phases/) — per-phase and per-subphase requirement and implementation docs

## Status

Currently in **Phase 0 — Requirements Collection**. See [`docs/phases/phase0/PHASE0.md`](docs/phases/phase0/PHASE0.md).

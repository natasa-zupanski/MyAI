# Phase Documentation Guide

Reference for generating and maintaining phase/subphase MD files in this project.

---

## Folder Structure

```
docs/
  PLAN.md                        # overall project plan, all phases at a glance
  PHASE-DOC-GUIDE.md             # this file
  phases/
    phase0/                      # one folder per phase
      PHASE0.md                  # phase overview
      PHASE0-1-hardware.md       # subphase files live alongside overview
      PHASE0-2-serving.md
      ...
    phase1/
      PHASE1.md
      PHASE1-1-model-setup.md
      ...
  private/                       # gitignored — never committed (see below)
    hardware.md
    ...
README.md                        # project root — stays at repo root, not in docs/
```

### Principles

- `docs/` holds all documentation. No MD files live at the repo root except `README.md`.
- `docs/phases/phase{N}/` holds the overview and all subphase files for phase N. Files for the same phase are siblings in one folder.
- `PLAN.md` and `PHASE-DOC-GUIDE.md` live directly in `docs/` because they are project-wide, not tied to a single phase.
- Create `docs/phases/phase{N}/` when the phase overview is first written. Do not create the folder speculatively before the overview exists.

---

## Private Files (`docs/private/`)

`docs/private/` is listed in `.gitignore` and is never committed to the public repo. Use it for any information that should inform development but must not be public.

### What belongs in `docs/private/`

| File | Contents |
|---|---|
| `hardware.md` | Exact hardware specs (GPU model, RAM, CPU, storage) |
| `env.md` | Local paths, environment variable values, port assignments |
| `credentials.md` | API keys, tokens, local service passwords |
| `context.md` | Personal project context, constraints, or preferences not suitable for a public README |

Add new files as needed. One file per topic; keep names short and lowercase.

### How to reference private files from public phase docs

- In the public phase doc, replace specific private details with a summary line (e.g., "4 GB VRAM — see private file") and a link: `[docs/private/hardware.md](../../private/hardware.md)`
- Never copy private values back into a public doc, even in comments
- The private file is the single source of truth for that data; the public doc references it

### When working with Claude Code

Claude Code can read `docs/private/` files to inform implementation decisions during a session. Sensitive values there do not need to be re-stated in the conversation — just point to the file.

### Link paths between folders

| Link type | Example path |
|---|---|
| Subphase → sibling subphase (same phase) | `PHASE0-2-serving.md` |
| Subphase → next phase overview | `../phase1/PHASE1.md` |
| Phase overview → its own subphase | `PHASE0-1-hardware.md` |
| Phase overview → next phase | `../phase1/PHASE1.md` |
| Any phase doc → PLAN or guide | `../../PLAN.md` |

All links are relative. Never use absolute paths or repo-root-relative paths.

---

## File Naming Convention

### Overview files
```
PHASE{N}.md
```
One per phase. Contains purpose, subphase index table, completion criteria, and upstream dependencies.
Example: `PHASE0.md`, `PHASE1.md`

### Subphase files
```
PHASE{N}-{M}-{topic}.md
```
- `{N}` — phase number (0, 1, 2, ...)
- `{M}` — subphase number within that phase (1, 2, 3, ...)
- `{topic}` — lowercase hyphenated short name describing the subphase content

Examples:
- `PHASE0-1-hardware.md`
- `PHASE0-3-api-interface.md`
- `PHASE1-2-server-config.md`

**Rule:** Topic names should be short (1–3 words), unambiguous, and grep-friendly. Avoid generic names like `notes` or `misc`.

---

## Required Sections per File

### Overview file (`PHASE{N}.md`)
1. `# Phase {N} — {Title}` heading
2. `## Status:` — one of: `Not Started`, `In Progress`, `Complete`
3. `## Purpose` — one paragraph explaining the goal of this phase
4. `## Subphases` — markdown table with columns: File (linked), Topic, Status
5. `## Completion Criteria` — bulleted list of what "done" means for this phase
6. `## Dependencies` — which phases must be complete before this one starts

### Subphase file (`PHASE{N}-{M}-{topic}.md`)
1. `# Phase {N}-{M} — {Full Title}` heading
2. `## Status:` inline on same line (e.g., `## Status: Pending`)
3. Body sections — vary by content; use `##` for major sections, `###` for subsections
4. `## Checklist` — `- [ ]` items for all open decisions/actions; check off completed ones with `- [x]`
5. `## Key Decisions` — table with columns: Decision, Choice, Notes; populate `Choice` with `TBD` until resolved, then fill in
6. `## Next` — one line pointing to the next file in sequence with a relative markdown link

---

## Status Values

Use exactly these strings (for grep consistency):

| Value | Meaning |
|---|---|
| `Not Started` | Phase 0 complete, but work hasn't begun |
| `Pending` | Blocked on a previous subphase or decision |
| `In Progress` | Actively being worked on |
| `Complete` | All checklist items checked, all Key Decisions filled |

---

## Conventions Decided During Initial Generation

### Linking
- Always use relative markdown links between phase docs: `[PHASE1.md](PHASE1.md)`
- The `## Subphases` table in overview files links to subphase files
- Each subphase's `## Next` section links to the next subphase or the next phase overview

### Checklist items
- Use `- [ ]` for open items
- Use `- [x]` for confirmed/completed items
- Items prefixed with `**TODO:**` are action items requiring user input, not implementation tasks
- Items without `**TODO:**` are implementation tasks to be done during that phase

### Key Decisions table
- `Choice` column is `TBD` until decided; replace with the actual decision when locked
- `Notes` column explains rationale or constraints briefly
- Once a decision is made, it is not changed in this table — create a new row or append a note if it changes

### Options tables (comparison sections)
- Use a `| Property | Notes |` two-column table for comparing a single option's attributes
- Use a wider table with one row per option when comparing multiple options side-by-side

### Recommendations
- If a clear recommendation can be made given available information, state it explicitly in a `## Recommendation` section
- Prefix with the constraint it depends on: "Given the Quadro T1000 Max-Q (4 GB VRAM): ..."
- Mark as pending if the underlying hardware/requirement decision is not yet locked

### Draft content
- If a phase doc contains draft content (e.g., a draft system prompt), wrap it in a fenced code block and label it `Draft` in the surrounding prose
- Note explicitly what needs to be refined and when (e.g., "refine in Phase 3")

### Hardware-specific notes
- Hardware decisions flow from `PHASE0-1-hardware.md` and are referenced (not repeated) in downstream docs
- Use phrasing like "Given [hardware detail]:" to make the dependency explicit

### Phase doc for future/deferred phases
- Create the overview `PHASE{N}.md` immediately when the phase is planned, even if implementation is far off
- Set status to `Not Started` and leave subphase files as stubs or create them when the phase becomes active
- Do not create subphase files for future phases until Phase 0 decisions that affect them are locked — avoids documenting choices that will change

---

## File Creation Order

When a new phase is planned:
1. Create `PHASE{N}.md` first (overview)
2. Create each `PHASE{N}-{M}-{topic}.md` in subphase order
3. Update `PLAN.md` to reference new phase files if the overall plan changes

When a decision is made during a conversation:
1. Find the relevant subphase file
2. Check the checklist item (`- [x]`)
3. Fill in the `Key Decisions` table row

---

## Example: Adding a New Subphase

If Phase 2 needs a new subphase for "logging":
- Filename: `PHASE2-5-logging.md`
- Add a row to the `## Subphases` table in `PHASE2.md`
- Previous subphase's `## Next` line updated to point to this file
- This file's `## Next` points to the next file after it

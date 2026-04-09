[Documentation Index](../index.md) / commands / plan

# plan — Implementation Plan Creator

**Slash Command:** `/plan`
**Source File:** `commands/plan.md`
**Type:** Multi-step — reads codebase, asks questions, writes plan files

---

## Table of Contents

- [Purpose](#purpose)
- [Plan Folder Structure](#plan-folder-structure)
- [The Single-Session Principle](#the-single-session-principle)
- [Step 0 — Load Project Context](#step-0--load-project-context)
- [Step 1 — Pre-Planning Questions](#step-1--pre-planning-questions)
- [Step 2 — Codebase Analysis & Risk Assessment](#step-2--codebase-analysis--risk-assessment)
- [Step 3 — Create Plan Folder and Files](#step-3--create-plan-folder-and-files)
- [Step 4 — Implementation Sessions Structure](#step-4--implementation-sessions-structure)
- [Step 5 — EXECUTE.md Structure](#step-5--executemd-structure)
- [Step 6 — Final Documentation Update Session](#step-6--final-documentation-update-session)
- [Mandatory Constraints](#mandatory-constraints)
- [Execution Violations](#execution-violations)

---

## Purpose

`/plan` guides Claude Code through creating a thorough, multi-phase implementation plan before writing any code. It enforces a structured pre-planning phase (questions → codebase read → risk analysis) before producing a plan folder with execution instructions.

The plan is then executed session-by-session using `EXECUTE.md` — one bounded unit of work at a time, always waiting for user approval before proceeding.

---

## Why This Command Exists

The core problem `/plan` solves is mid-execution drift. When an AI starts writing code before fully understanding the problem, any incorrect assumption made in the first few files propagates forward. By the time the error surfaces, the session is long, context has been compacted, and stopping to correct course consumes more tokens than the original task. The longer the correction attempt, the worse the drift gets.

`/plan` front-loads all understanding into a read-only, write-nothing session. The output is a set of markdown files — durable, editable, and completely outside the context window — that serve as the AI's ground truth for every subsequent execution session. Any misunderstanding caught in the plan costs a sentence to fix. The same misunderstanding caught at session 10 of 12 costs the entire session and likely the coherence of the conversation.

---

## Plan Folder Structure

Plans are created at `plans/{kebab-case-subject}-plan/`:

```
plans/
└── {kebab-case-subject}-plan/
    ├── 01-current-state.md           — current codebase state relevant to the task
    ├── 02-problems-and-caveats.md    — risks, edge cases, known issues
    ├── 03-implementation-sessions.md — all sessions with checklists
    └── EXECUTE.md                    — session-by-session execution guide
```

---

## The Single-Session Principle

> [!WARNING]
> This is the hardest constraint in `/plan`. Violating it is a hard failure.

When the user references or opens any file named `EXECUTE.md`, Claude Code must:

1. **Stop. Do NOT read `EXECUTE.md`.**
2. Read ONLY `03-implementation-sessions.md`.
3. Find the first session with at least one unchecked `[ ]` item — that is the current session.
4. Announce only that session number and title. Present its diff as markdown code blocks in chat. **Do not use file modification tools yet.**
5. End your response by asking: "Approve this session?"
6. Only after the user explicitly approves: use tools to modify files for that session only.
7. After executing: update the `03-implementation-sessions.md` checklist, then **hard stop**. End with "Session N complete. Ready when you are." — nothing more.
8. When the user says "next", "continue", "proceed", "approved", or "done" — re-read `03-implementation-sessions.md`, find the next incomplete session, and repeat from step 3.

Do NOT mention future sessions, remaining session counts, or what comes next at any point.

---

## Step 0 — Load Project Context

Before anything else:

1. Read `docs/aicontext.md` — global rules, terminology, tech stack, directory mapping
2. Use the Directory Mapping to identify which domain(s) this task touches
3. Read the domain-specific `aicontext.md` if one exists
4. Read `docs/doc-maintenance/documentationMaintenanceGuide.md` — formatting rules and changelog procedures

### Code-Review-Graph Dependency Analysis (Step 0, after loading context)

5. Identify the entry-point file(s) for the task (the main service, controller, or route being modified)
6. Call the code-review-graph MCP tools in this order:
   - `get_minimal_context(task="<task description>")` — surfaces highest-value files and communities
   - `get_impact_radius(changed_files=["<entry-point>"], detail_level="minimal")` — risk-scored blast radius
   - `query_graph(pattern="importers_of", target="<entry-point>", detail_level="minimal")` — direct dependents
7. Use the output to build a "files to examine" list — read only those files in Step 2, skip unrelated files
8. Record the graph output in `01-current-state.md` under a "Dependency Analysis" section

> [!WARNING]
> Never use `code-review-graph detect-changes` here — it reads git-staged diffs only. Use the MCP tools above.

---

## Step 1 — Pre-Planning Questions

Ask all questions below and **wait for complete answers before proceeding to Step 2**.

### 1.1 — Scope of Files

- Which existing source files are in scope? List every file that will be touched: schemas, services, controllers, routes, middleware, validators, utilities
- Is this modifying an existing feature, or entirely new functionality?
- If modifying: provide exact file paths

### 1.2 — Route Validators & Middleware

- Does this task require new request validators?
- Should validators go in a new file or an existing one?
- Are there new middleware or authentication guards required?
- Are there new routes being added to any router file?

### 1.3 — Logic Deep-Dive

Minimum questions to ask:

- What is the exact happy-path flow, step by step?
- What are the failure and edge cases, and how should each be handled?
- What data dependencies exist — which schemas are read, written, or referenced?
- Are there race conditions or concurrency concerns?
- Does this affect any existing flows or shared services?
- Are there external integrations involved?
- What are the specific error messages and HTTP status codes for each failure case?
- Are there email notifications — if so, which SMTP path?
- Does this touch any modularised service sub-modules?

### 1.4 — Documentation Impact

- Are any API endpoints changing (added, modified, removed)?
- Are any database schemas changing?
- Are any workflows or processes changing?
- Are any environment variables changing?
- Which documentation files will need updating after implementation?

---

## Step 2 — Codebase Analysis & Risk Assessment

Before writing any plan file:

1. Read files identified by the graph in Step 0 — prioritise high-risk dependents first, skip files not in the graph output
2. Read the relevant domain docs to avoid contradictions
3. Perform risk analysis across:
   - Security (auth bypass, injection, data leakage)
   - Data integrity (race conditions, partial writes, missing rollbacks)
   - Regression (shared services used by other features)
   - Logic (edge cases, null handling, invalid states)
   - Email/notification (fire-and-forget detachment, duplicate sends)

> [!NOTE]
> The graph output from Step 0 is the reading list for Step 2. Do not read entire directories — only read files the graph identified plus the baseline files (router, schema, controller, validator for the affected domain).

---

## Step 3 — Create Plan Folder and Files

Create `plans/{kebab-case-subject}-plan/` with all four files:

```
01-current-state.md
02-problems-and-caveats.md
03-implementation-sessions.md
EXECUTE.md
```

---

## Step 4 — Implementation Sessions Structure

`03-implementation-sessions.md` defines every session. Each session contains:

```markdown
## Session N — [Goal]

**Files changed:**
- `src/path/to/file.ts`

**Changes:**
1. [Numbered description of change]
2. [...]

**Verification:**
- [ ] [Single observable fact to verify]
- [ ] [...]
```

Verification items must be scoped to a single observable fact per item. No compound checks.

---

## Step 5 — EXECUTE.md Structure

`EXECUTE.md` is a **protocol trigger file only**. It must never contain code, file paths, session titles, diffs, or step listings. All plan content lives exclusively in `03-implementation-sessions.md`.

The file must be 5 lines maximum. Use this exact template — nothing else:

```markdown
# EXECUTE — {Subject}

> Read `03-implementation-sessions.md`.
> Find the first session with an unchecked `[ ]` item.
> Present only that session's diff. Wait for approval.
> After executing and updating the checklist, stop and wait.
> User says "next" to advance.
```

---

## Step 6 — Final Documentation Update Session

The last session in every plan is a documentation update session:

- Update all affected `docs/` files identified in Step 1.4
- Update `docs/doc-maintenance/changelog.json` (prepend newest entry)
- Update `docs/doc-maintenance/changelog.md` (prepend newest row)

---

## Mandatory Constraints

- Only use the project's defined tech stack — never introduce new dependencies
- Follow existing modular service patterns — do not create monolithic service files
- No fragmented docs — edit existing files only, never create `v2` or `notes` variants
- Use consistent terminology from `docs/aicontext.md` throughout the plan

---

## Execution Violations

The following are hard failures during plan execution:

| Violation | Description |
|-----------|-------------|
| Reading EXECUTE.md for content | Reading `EXECUTE.md` for session details instead of reading `03-implementation-sessions.md` directly |
| Previewing future sessions | Describing or proposing changes for any session beyond the current one |
| Batching sessions | Executing more than one session in a single response |
| Reading irrelevant files | Reading files only needed for future sessions |
| Using file tools before approval | Using any file modification tool before the user explicitly approves the proposed diff |
| Out-of-scope edits | Making any change not explicitly listed in the approved session |
| Continuing after hard stop | Describing or proposing the next session in the same response as a checklist update |

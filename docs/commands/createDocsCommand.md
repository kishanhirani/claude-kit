[Documentation Index](../index.md) / commands / create-docs

# create-docs — Documentation Creator

**Slash Command:** `/create-docs`
**Source File:** `commands/create-docs.md`
**Type:** Multi-step — Q&A phase, codebase inventory, gap analysis, session-driven plan folder

---

## Table of Contents

- [Purpose](#purpose)
- [When to Use](#when-to-use)
- [Allowed Tools](#allowed-tools)
- [Constraints](#constraints)
- [How It Works](#how-it-works)
- [Pre-Planning Q&A](#pre-planning-qa)
- [Plan Folder Structure](#plan-folder-structure)
- [Session Granularity](#session-granularity)
- [The Single-Session Principle](#the-single-session-principle)
- [Step-by-Step Process](#step-by-step-process)
- [Output: Plan Report](#output-plan-report)
- [Execution Violations](#execution-violations)

---

## Purpose

`/create-docs` is a planning skill for codebases that have no documentation yet (or incomplete documentation). It does not write documentation directly — instead, it inventories the codebase, identifies every documentation gap, and produces a structured plan folder at `plans/create-docs-{YYYY-MM-DD}-plan/`.

That plan is then executed session by session using `EXECUTE.md`, following the same Single-Session Principle as `/plan`. This allows documentation to be created across many conversations without losing state — each session is small, bounded, and tracked by a checklist.

---

## When to Use

- A codebase has skills in `commands/` with no corresponding docs in `docs/commands/`
- Documentation exists but is incomplete, stale, or out of sync with source files
- A large codebase needs documentation produced incrementally over many sessions
- After a burst of feature development that has outpaced documentation

---

## Allowed Tools

```
Read
Write
Glob
Bash(git log*)
Bash(git status*)
```

Only read and plan-writing operations are permitted during the `/create-docs` run itself. All actual documentation writing happens during `EXECUTE.md` execution.

---

## Constraints

- **Do NOT** modify any source code files — documentation and plan files only
- **Do NOT** write documentation content during the `/create-docs` run — only conduct the Q&A and create the plan folder
- **Do NOT** proceed past the Q&A phase until the user has answered all questions
- Every planned documentation file must follow the breadcrumb, heading, and formatting rules defined in the skill's inlined standards (Step 0 of the skill file) — the skill does not depend on `docs/` files existing
- Never create "v2", "notes", or "update" variant files — one canonical file per topic only
- Never batch multiple documentation files into one session

---

## How It Works

```
/create-docs
     │
     ├── Step 0: Load context (reads any existing docs/ files; falls back to inlined rules)
     ├── Step 1: Q&A — asks user for project identity, scope, style, and priorities
     │           STOPS and WAITS for answers before proceeding
     ├── Step 2: Inventory codebase (skills, existing docs, source structure)
     ├── Step 3: Identify all gaps (new-skill-doc, aicontext-drift, index-drift, etc.)
     ├── Step 4: Create plan folder at plans/create-docs-{date}-plan/
     │           (01-inventory.md includes Q&A answers for future session context)
     ├── Step 5: Write 03-documentation-sessions.md (one session per gap, priority-ordered)
     ├── Step 6: Write EXECUTE.md (runtime guide, Single-Session Principle enforced)
     ├── Step 7: Append final changelog session
     └── Step 8: Report summary to user

     Then separately:
     User opens EXECUTE.md → works through sessions one at a time
```

---

## Pre-Planning Q&A

Before reading any code, `/create-docs` asks the user a set of questions in a single message and waits for complete answers. This ensures the plan is shaped by the user's actual intentions — not guessed from the codebase alone.

Questions are grouped into five areas:

| Group | What Is Asked |
|---|---|
| **Project identity** | Name, purpose, tech stack, intended documentation audience |
| **Documentation scope** | Which parts of the codebase to document, what to exclude, whether to include a getting started guide and workflow guides |
| **Structure preferences** | Docs directory location, whether to include `aicontext.md` files, files to preserve as-is |
| **Style preferences** | Tone, depth of code examples, existing naming conventions to follow |
| **Prioritisation** | Which sections are highest priority, any deadline driving the effort |

The answers are recorded in `01-inventory.md` inside the plan folder. Every session executed from `EXECUTE.md` can reference these answers to stay consistent with the user's stated intentions — even sessions run weeks later in a new conversation.

> [!NOTE]
> `/create-docs` will not proceed to inventory the codebase until all Q&A answers have been received.

---

## Plan Folder Structure

```
plans/
└── create-docs-{YYYY-MM-DD}-plan/
    ├── 01-inventory.md               — snapshot of every skill and every gap found
    ├── 02-gaps-and-scope.md          — every gap with type, affected file, and resolution
    ├── 03-documentation-sessions.md  — all sessions with checklists (single source of truth)
    └── EXECUTE.md                    — session-by-session execution guide (no checklists)
```

---

## Session Granularity

Each session covers exactly one gap. The sessions are kept small deliberately so that work can be distributed across many conversations without losing state.

| Gap Type | Session Size |
|---|---|
| New skill doc | 1 session per skill |
| Incomplete skill doc fix | 1 session per skill |
| `docs/commands/readme.md` + `docs/index.md` sync | 1 session (always together) |
| `aicontext.md` sync | 1 session per aicontext file |
| Stale skill doc removal | 1 session |
| Changelog entries (json + md) + progress tracker | Always the final session |

---

## The Single-Session Principle

> [!WARNING]
> This is the hardest constraint in `EXECUTE.md` execution. Violating it is a hard failure.

When executing the plan via `EXECUTE.md`, Claude Code must:

1. Read `03-documentation-sessions.md` and identify the **first session whose checklist is not fully marked `[x]`**
2. Announce **that session only** — never preview or describe future sessions
3. Present the proposed documentation content as markdown in the chat response (no file writes yet)
4. Ask a yes/no approval question, then **wait**
5. Only after explicit approval: use Write or Edit tools to create or modify the documentation files for **that session only**
6. After execution: update the `03-documentation-sessions.md` checklist for that session
7. **Stop completely** and wait for the user to confirm before proposing the next session

---

## Step-by-Step Process

### Step 0 — Load Context

Read any of the following that exist (they may not — that is normal for a greenfield project):

1. `docs/aicontext.md`
2. `docs/index.md`
3. `docs/commands/readme.md`

If none exist, the skill proceeds using its inlined documentation standards only.

### Step 1 — Pre-Planning Q&A

Ask the user all questions in one message and wait for complete answers before proceeding. Questions cover:

- **Project identity** — name, purpose, tech stack, intended audience
- **Documentation scope** — which parts of the codebase to document, what to exclude, whether to include a getting started guide and workflow guides
- **Structure preferences** — docs directory location, whether to include `aicontext.md` files, files to preserve
- **Style preferences** — tone, depth of code examples, naming conventions
- **Prioritisation** — which sections are highest priority, any deadline driving the effort

The answers are recorded in `01-inventory.md` inside the plan folder so every future session has the context it needs to stay consistent with the user's intentions.

### Step 2 — Inventory the Codebase

1. Glob `commands/*.md` and read each skill file — record name, frontmatter, and prompt body
2. List all existing `docs/` files recursively
3. Cross-reference against `docs/index.md` and `docs/commands/readme.md` (if they exist) to find what is missing, stale, or incomplete
4. Read the project root `readme.md` and run `git log --oneline -20` for project context
5. If source code is in scope: list top-level contents of each in-scope folder

### Step 3 — Identify All Documentation Gaps

Classify each gap as one of:

| Gap Type | Description |
|---|---|
| `new-skill-doc` | Skill in `commands/` has no doc file |
| `missing-section` | A `docs/` section is missing `readme.md` or `aicontext.md` |
| `stale-skill-doc` | Doc file exists for a skill that no longer exists in `commands/` |
| `incomplete-skill-doc` | Doc file exists but is missing required sections |
| `aicontext-drift` | `aicontext.md` is missing new skills, has stale entries, or outdated terminology |
| `index-drift` | `docs/index.md` or `docs/commands/readme.md` does not match current state |
| `changelog-missing` | Changelog has no entry for changes already made |

### Step 4 — Create the Plan Folder

Create `plans/create-docs-{YYYY-MM-DD}-plan/` with four files: `01-inventory.md` (includes Q&A answers), `02-gaps-and-scope.md`, `03-documentation-sessions.md`, `EXECUTE.md`.

### Step 5 — Write `03-documentation-sessions.md`

One session per gap, ordered by priority (high-priority gaps from Step 1.5 come first). Each session block:

```markdown
## Session N — {Gap Type}: {Short Description}

**Goal:** One sentence.
**Files created or modified:**
- `docs/path/to/file.md`

### Changes

1. {Specific change}

### Verification

**`docs/path/to/file.md`**
- [ ] {Single observable fact}
- [ ] {Single observable fact}
```

### Step 6 — Write `EXECUTE.md`

Session-by-session runtime guide. No inline checklists — all tracking is in `03-documentation-sessions.md`. Every session ends with a mandatory checklist update block before the user is asked to verify.

### Step 7 — Final Session: Changelog

The last session always updates:
- `docs/doc-maintenance/changelog.json` — prepend new entry
- `docs/doc-maintenance/changelog.md` — prepend new row
- `docs/doc-maintenance/documentationProgress.md` — update all rows and "Last Updated" date

---

## Output: Plan Report

After creating the plan folder, `/create-docs` outputs:

```
Plan created: plans/create-docs-{YYYY-MM-DD}-plan/

Gaps found:
- {N} new skill docs to create
- {N} incomplete skill docs to fix
- {N} aicontext.md files to sync
- {N} index/readme drift fixes
- {N} stale skill docs to remove

Sessions planned: {total}
Estimated size: {small | medium | large} ({N} files to create or modify)

To begin: open EXECUTE.md and start Session 1.
```

---

## Execution Violations

The following are hard failures during `EXECUTE.md` execution:

| Violation | Description |
|---|---|
| Previewing future sessions | Describing or proposing content for any session beyond the current one |
| Batching sessions | Writing documentation for more than one session in a single response |
| Writing before approval | Using Write or Edit tools before the user explicitly approves the proposed content |
| Reading irrelevant files | Reading files only needed for future sessions |
| Out-of-scope writes | Creating or modifying files beyond the approved session |
| Scope creep | Adding content not specified in the session (e.g. fixing an adjacent issue "while here") |

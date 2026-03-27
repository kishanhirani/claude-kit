---
description: Create full documentation for an existing codebase that has none — produces a session-by-session plan that can be executed across many conversations.
allowed-tools: Read, Write, Glob, Bash(git log*), Bash(git status*)
---

# Documentation Creator

**Important constraints — follow these strictly:**
- Do NOT modify any source code files — documentation and plan files only.
- Do NOT create documentation content during this skill run — only conduct the Q&A and create the plan folder and its files.
- Do NOT proceed past the Q&A (Step 1) until the user has answered all questions.
- Follow the inlined documentation standards in Step 0 of this skill — do not depend on external docs/ files existing.
- Every documentation file you plan to create must follow the breadcrumb, heading, and formatting rules defined in Step 0.
- Never create "v2", "notes", or "update" variant files — one canonical file per topic only.
- Documentation is written and tracked session-by-session via `EXECUTE.md`. Never batch multiple doc files into one session.

---

## Step 0 — Load Context

Read any of the following that exist (they may not — that is normal, this skill creates them):

1. `docs/aicontext.md` — global rules, key terminology, writing style, directory mapping, critical system behaviours.
2. `docs/index.md` — current documentation structure and all active links.
3. `docs/commands/readme.md` — current commands section state.

If none of these exist, proceed from the rules inlined in this skill only. Do not abort.

### Inlined Documentation Standards (always apply regardless of what exists in docs/)

**File structure rules:**
- Every markdown file must start with a breadcrumb: `[Documentation Index](../index.md) / [section] / [topic]` — or `[Documentation Index](./index.md)` for root-level files.
- Use semantic headers (`#`, `##`, `###`, `####`).
- Skill documentation must include: purpose, allowed tools, constraints, step-by-step process, output format.

**Naming conventions:**
- Folder names: camelCase (e.g. `gettingStarted/`, `doc-maintenance/`)
- File names: camelCase (e.g. `aicontext.md`, `commitCommand.md`, `syncGuide.md`)
- Exception: `index.md` and `readme.md` at the root of each folder remain lowercase

**Formatting rules:**
- Use Mermaid.js (`sequenceDiagram`, `flowchart`) for flows and architecture diagrams
- Wrap all code in triple backticks with correct language tag (`bash`, `typescript`, `json`, `markdown`)
- Use markdown callouts (`> [!WARNING]`, `> [!NOTE]`) for critical constraints
- Never expose real API keys, tokens, credentials, or PII in examples — use `[REDACTED_API_KEY]`, `[USER_ID]`, `[BRANCH_NAME]` placeholders

**Changelog format (for the final session):**
- `docs/doc-maintenance/changelog.json` — prepend a new object: `{ "session": N, "date": "YYYY-MM-DD", "focus": "...", "files": [...], "summary": "..." }`
- `docs/doc-maintenance/changelog.md` — prepend a new table row: `| N | YYYY-MM-DD | focus phrase | summary |`
- `docs/doc-maintenance/documentationProgress.md` — update "Last Updated" date and all affected rows

**Never do:**
- Create "v2", "notes", or "update" suffixed files — edit existing files only
- Skip updating the progress tracker after a documentation session
- Skip updating the changelog after a documentation session

---

## Step 1 — Pre-Planning Q&A

Before reading any code or creating any files, ask the user ALL of the questions below and wait for complete answers. Do NOT proceed to Step 2 until every question has been answered.

Present the questions grouped as shown. Ask them all in one message — do not drip-feed them one at a time.

### 1.1 — Project Identity

- What is the project name and what does it do? (One paragraph is enough.)
- What is the primary tech stack? (Language, frameworks, key libraries.)
- Who is the intended audience for the documentation? (e.g. internal team, open-source contributors, onboarding developers, AI assistants.)

### 1.2 — Documentation Scope

- Which parts of the codebase should be documented? (e.g. all of `commands/`, a specific `src/` subfolder, everything, or a named subset.)
- Are there any files, folders, or topics that should be explicitly excluded?
- Should the documentation include a getting started / setup guide? If yes, what prerequisites and installation steps are involved?
- Should the documentation include workflow guides — patterns for how features or commands are combined and used together?

### 1.3 — Documentation Structure Preferences

- Where should documentation live? (Default: `docs/` at the project root. Override if the project uses a different convention.)
- Should each docs section have an `aicontext.md` file — an AI-readable context file that gives future AI assistants domain-specific terminology and constraints? (Recommended: yes.)
- Are there existing documentation files that must be preserved as-is and not overwritten?

### 1.4 — Style and Format Preferences

- What tone should the documentation use? (e.g. technical and terse, conversational, step-by-step tutorial style.)
- Should code examples be included in skill or API documentation? If yes, how detailed?
- Any naming conventions already in use for files or folders that should be followed?

### 1.5 — Prioritisation

- If the full documentation requires many sessions, which sections are highest priority and should be planned as early sessions?
- Is there a deadline or milestone driving this documentation effort?

---

## Step 2 — Inventory the Codebase

Use the Q&A answers from Step 1 to scope the inventory — only examine the parts of the codebase the user confirmed are in scope.

### 2.1 — Skill Inventory

1. Glob `commands/*.md` to list all skill files.
2. For each skill file, read it and record:
   - Skill name (filename without extension)
   - `description` from frontmatter
   - `allowed-tools` from frontmatter
   - All constraints, steps, and output format rules from the prompt body
3. Cross-reference against `docs/index.md` and `docs/commands/readme.md` (if they exist) to identify:
   - Skills with no documentation file yet
   - Skills whose documentation is missing or incomplete
   - Skills documented but no longer present in `commands/`

### 2.2 — Documentation Structure Inventory

1. List all existing files under the docs directory (default `docs/`) recursively.
2. For each existing section folder, note:
   - Which `aicontext.md` files exist and which are missing (skip if user opted out in Step 1.3)
   - Which `readme.md` files exist and which are missing
   - Which skill doc files exist vs what the skills table lists

### 2.3 — Codebase Context

1. Read the project's root `readme.md` (if present).
2. Run `git log --oneline -20` to understand recent activity.
3. Note the top-level directory structure — this informs which `docs/` sections need creating.
4. If source code exists beyond `commands/` (e.g. `src/`, `lib/`, `app/`) and the user confirmed it is in scope: list the top-level contents of each in-scope folder.

---

## Step 3 — Identify All Documentation Gaps

Produce a complete gap list by comparing the inventory from Step 2 against the existing docs state. Apply the scope and exclusions the user defined in Step 1.2 — do not flag gaps for out-of-scope areas.

For each gap, classify it as one of:

| Gap Type | Description |
|---|---|
| `new-skill-doc` | A skill in `commands/` has no documentation file in `docs/commands/` |
| `missing-section` | A `docs/` section folder needs a `readme.md` or `aicontext.md` that does not exist |
| `stale-skill-doc` | A documentation file describes a skill that no longer exists in `commands/` |
| `incomplete-skill-doc` | A documentation file exists but is missing required sections (purpose, allowed-tools, steps, output format, constraints) |
| `aicontext-drift` | An `aicontext.md` file is missing new skills, stale entries, or outdated terminology |
| `index-drift` | `docs/index.md` or `docs/commands/readme.md` does not match current state |
| `changelog-missing` | `docs/doc-maintenance/changelog.json` or `changelog.md` has no entry for changes already made |

---

## Step 4 — Create the Plan Folder

Create `plans/create-docs-{YYYY-MM-DD}-plan/` with these four files. Use the Q&A answers from Step 1 to populate `01-inventory.md` — record the user's stated project identity, scope decisions, style preferences, and priority order alongside the technical inventory. This context must be visible in the plan so future sessions are consistent with the user's intentions.

### Required Files

| File | Purpose |
|---|---|
| `01-inventory.md` | Full snapshot: every skill found, every gap identified, current docs state, and Q&A answers from Step 1 |
| `02-gaps-and-scope.md` | Every gap from Step 3 listed with its type, affected file, and resolution |
| `03-documentation-sessions.md` | One session per gap — each small, self-contained, and verifiable |
| `EXECUTE.md` | Session-by-session execution guide — no checklists, just steps; Single-Session Principle enforced |

### File Header (every file)

```markdown
# {Title}

**Plan:** create-docs-{YYYY-MM-DD}
**File:** {filename}
**Created:** {YYYY-MM-DD}

---
```

---

## Step 5 — Structure of `03-documentation-sessions.md`

Each session covers exactly one gap. Never combine two unrelated gaps into one session.

Session size guidelines:
- One new skill doc = one session
- One incomplete skill doc fix = one session
- `docs/commands/readme.md` + `docs/index.md` table sync = one session (they always change together)
- One `aicontext.md` sync = one session
- Changelog entries (json + md) = always the final session

Each session block:

```markdown
## Session N — {Gap Type}: {Short Description}

**Goal:** One sentence describing what this session produces.
**Files created or modified:**
- `docs/path/to/file.md`

### Changes

1. {Specific change description}
2. {Specific change description}

### Verification

**`docs/path/to/file.md`**
- [ ] {Single observable fact — e.g. "file exists with correct breadcrumb"}
- [ ] {Single observable fact — e.g. "skills table has row for /create-docs"}
```

Verification rules:
- Each `- [ ]` must be one observable fact scoped to the specific change
- Aim for 1–4 items per file group
- `03-documentation-sessions.md` is the single source of truth for checklist progress — `EXECUTE.md` contains no checklists

---

## Step 6 — Structure of `EXECUTE.md`

`EXECUTE.md` is the runtime guide for executing the plan. It must:

- Reference each session from `03-documentation-sessions.md` as a top-level section
- Break each change to the **smallest independently verifiable unit** (one section of one file at a time)
- State prerequisites at the top of each session
- Contain **no inline checklists** — all tracking lives in `03-documentation-sessions.md`
- End every session with a mandatory checklist update block

### EXECUTE.md Execution Rules (enforced at runtime — never violate these)

> **THE SINGLE-SESSION PRINCIPLE:** The AI's entire mental scope during execution is ONE session. It must not read ahead, plan ahead, or acknowledge the existence of any session beyond the current one.

When a user opens `EXECUTE.md`, the AI must follow these rules without exception:

0. **Identify the current session first.** Read `03-documentation-sessions.md` and find the first session whose checklist is not fully marked `[x]`. That is the current session. Announce only that session number and title. Do not mention any other session.
   - **CRITICAL:** At this stage, output the proposed documentation content as markdown in the chat response. Do NOT use file write tools yet.
1. **One session at a time — strictly enforced.** Present the proposed content for the **current session only**. Never present content for Session N+1 alongside Session N.
   - Do NOT read files only relevant to future sessions.
   - Do NOT mention what future sessions will produce, even as a preview.
   - Do NOT say "there are N sessions remaining" or "next we will…"
   - The response must end after presenting the current session and asking for approval. Nothing else.
2. **Stop and wait.** After presenting the session's content, ask a single yes/no question: approve or adjust. Do NOT write any files until the user explicitly approves.
   - ONLY AFTER APPROVAL: use Write or Edit tools to create or modify the documentation files for that session.
   - After approval, execute ONLY the changes for the approved session.
3. **No pre-emptive batching.** Sessions must be proposed individually regardless of size. Batching is forbidden.
4. **Checklist update before handoff.** After executing the tools, update `03-documentation-sessions.md` by marking each checklist item `[x]` before presenting the next session.
5. **Hard stop after checklist update.** After updating the checklist, stop generating text and wait for the user to reply. Your message ends with something like: "Session N complete. Checklist updated. Ready when you are." — nothing more.
6. **No scope creep within a session.** Write exactly what the session describes — nothing more. If you notice something adjacent that needs fixing, note it as a new gap — do not act on it now.
7. **Violation is a hard failure.** If these rules are not followed, the execution has failed.

### EXECUTE.md Template

```markdown
# EXECUTE — create-docs-{YYYY-MM-DD}

> Work through this file session by session.
> After completing each session, update the checklist in `03-documentation-sessions.md` before asking the user to verify.
> Identify the current session by reading `03-documentation-sessions.md` and finding the first session with incomplete checklist items.

---

## Session 1 — {Gap Type}: {Short Description}

**Prerequisites:** None
**Goal:** {one sentence}

### Step 1.1

**File:** `docs/path/to/file.md`
**Change:** {exact description of what to write}

---

> **Before asking the user to verify:** update the Session 1 checklist in `03-documentation-sessions.md` — mark each item `[x]` if done or `[-]` if skipped with a reason. A session is only complete once its checklist reflects the correct status.

---

## Session 2 — ...
```

---

## Step 7 — Final Session: Changelog

Always make the last session in `03-documentation-sessions.md` and `EXECUTE.md` a changelog update session.

This session:
1. Prepends a new entry to `docs/doc-maintenance/changelog.json`:
   ```json
   {
     "session": <next number>,
     "date": "YYYY-MM-DD",
     "focus": "create-docs run — initial documentation",
     "files": ["list of every file created or modified across all sessions"],
     "summary": "One dense sentence covering what was documented."
   }
   ```
2. Prepends a new row to `docs/doc-maintenance/changelog.md` (newest first).
3. Updates `docs/doc-maintenance/documentationProgress.md` — sets status and progress for all rows, updates "Last Updated" date.

---

## Step 8 — Report

After creating the plan folder, output a concise summary:

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

# Global AI Context: claude-kit

This file provides the primary context for any AI assistant interacting with the claude-kit documentation. It defines the project identity, technical standards, conventions, and documentation maintenance rules.

---

## 🏗 Project Identity & Architecture

- **Project Name:** claude-kit
- **Core Purpose:** A curated set of Claude Code skills, workflows, and documentation conventions that enable developers to achieve maximum and efficient output from Claude Code. This is a tooling and methodology repository — not a product backend.
- **Platform:** Claude Code CLI (Anthropic)
- **Documentation Format:** Markdown (CommonMark), Mermaid.js for diagrams
- **Repository Type:** Skills/commands toolkit — the `commands/` folder contains Claude Code skill definitions consumed by Claude Code's skill system.
- **Latest Anchored Commit:** `0f69116` — initial commit adding commands and readme.

---

## 📚 Documentation Standards

### Mandatory Reading

Before proposing any documentation changes, the AI **must** review:

- `docs/doc-maintenance/documentationMaintenanceGuide.md` — workflow, sync rules, and formatting standards
- `docs/index.md` — current documentation structure and all active links

### Key Project Terminology

- **Skill / Command:** A `.md` file in `commands/` that Claude Code loads as a slash command (e.g., `/commit`, `/plan`, `/daily-update`). Each skill has YAML frontmatter (`description`, `allowed-tools`) and a structured prompt body.
- **Slash Command:** The user-facing invocation form of a skill — `/commit` invokes `commands/commit.md`.
- **Session:** A single, bounded unit of Claude Code work defined within a plan's `EXECUTE.md`. Sessions execute one at a time; no session may begin until the previous one is confirmed complete.
- **Plan Folder:** A directory under `plans/` created by the `/plan` skill, containing `01-current-state.md`, `02-problems-and-caveats.md`, `03-implementation-sessions.md`, and `EXECUTE.md`.
- **Single-Session Principle:** The hard rule in `/plan` execution — Claude presents, waits for approval, executes, then stops. Never previews future sessions.
- **Daily Update:** The `/daily-update` skill that reads `git log` for the current day, filters cleanup noise, and outputs a concise done-list.
- **aicontext.md:** An AI-readable context file placed in each docs folder. It gives AI assistants domain-specific terminology, code paths, formatting rules, and strict constraints. Essential for AI-assisted documentation and code generation.
- **code-review-graph:** A globally installed CLI and MCP server that parses the codebase into a dependency graph. Used in `/plan`, `/create-docs`, and `/update-docs` to identify which files are affected by a change, their risk scores, and module community boundaries — replacing speculative full-directory reads with precision file selection. MCP tools are used during skill execution; the `update` CLI command is run after sessions complete to keep the graph current.

---

## ✍️ Writing Styles & Rules

### Naming Conventions

- All folder names: **camelCase** (e.g., `gettingStarted/`, `doc-maintenance/`)
- All file names: **camelCase** (e.g., `aicontext.md`, `commitCommand.md`, `syncGuide.md`)
- Exception: `index.md` and `readme.md` at the root of each folder remain lowercase

### Documentation Structure

- Every markdown file **must** start with a breadcrumb link: `[Documentation Index](../index.md) / [section] / [topic]`
- Root-level docs files use a direct path: `[Documentation Index](./index.md)`
- Use semantic headers (`#`, `##`, `###`, `####`)
- Skill references must include: purpose, usage syntax, allowed tools, step-by-step process, output format, and constraints

### AI Formatting Constraints

- **Diagrams:** Use Mermaid.js (`sequenceDiagram`, `flowchart`) for flows and architecture
- **Code:** Wrap all snippets in triple backticks with the correct language tag (`bash`, `typescript`, `json`, `markdown`)
- **Alerts:** Use markdown callouts (`> [!WARNING]`, `> [!NOTE]`) for critical constraints

---

## 🛠 Prohibitions (The "Never" List)

1. **NEVER** expose real API keys, tokens, or credentials in examples — use `[REDACTED_API_KEY]` or `[YOUR_TOKEN]`
2. **NEVER** create fragmented documentation — if a skill or feature changes, update the existing `.md` file, never create `v2` or `notes` variants
3. **NEVER** ignore the progress tracker — after any documentation session, update `docs/doc-maintenance/documentationProgress.md`
4. **NEVER** skip updating `docs/doc-maintenance/changelog.md` and `changelog.json` after a documentation session
5. **NEVER** use emoji in code or comments within skill `.md` files — only in documentation narrative sections where appropriate

---

## 🗺 Directory Mapping (Context Hints)

- **`docs/gettingStarted/`** → How to install, configure, and start using claude-kit; prerequisites; Claude Code setup
- **`docs/commands/`** → Documentation for each skill in `commands/` — commit, plan, daily-update; skill authoring guide
- **`docs/workflows/`** → End-to-end usage patterns combining multiple skills; best practices; efficiency guidelines
- **`docs/doc-maintenance/`** → Documentation maintenance guide, sync guide, progress tracker, changelogs, AI context update rules

---

## ⚙️ Critical System Behaviours (AI Must Know)

### Skill Frontmatter Format

Every skill in `commands/` requires YAML frontmatter at the top:

```markdown
---
description: One-line description used in skill picker
allowed-tools: Bash(git diff --cached*), Bash(git status*)
---
```

The `allowed-tools` field restricts which tools Claude Code can invoke when running the skill. Omitting it grants no tools. Wildcards (`*`) allow all sub-commands of a prefix.

### Code-Review-Graph Usage Across Skills

The `code-review-graph` MCP server is integrated into three skills. The rules below apply regardless of project:

| Skill | When to call | What to call |
|---|---|---|
| `/plan` | Step 0, before any file reads | `get_minimal_context`, `get_impact_radius`, `query_graph(pattern="importers_of")` |
| `/create-docs` | Step 2.0, before inventory | `get_minimal_context`, `get_impact_radius`, `query_graph(pattern="importers_of")` |
| `/update-docs` | Before identifying affected docs | `query_graph(pattern="importers_of")` on each modified file |

After any session that modifies files, run `code-review-graph update` (CLI) to keep the graph current.

**Never use `code-review-graph detect-changes`** inside skill execution — it reads git-staged diffs only and is only valid in git hook contexts (e.g. PreCommit).

Always start with `detail_level="minimal"`. Escalate to `"standard"` only if minimal output is insufficient.

### Single-Session Principle (Plan Execution)

When executing a plan created by `/plan`, Claude Code must:
1. Read `03-implementation-sessions.md` and identify the first unchecked session
2. Announce that session only — never preview future sessions
3. Present expected changes, ask for yes/no approval, then wait
4. Only after approval: execute that session's file changes
5. Update `03-implementation-sessions.md` checklist
6. Stop completely and wait for the user to confirm before proposing the next session

Violating this principle (batching sessions, previewing future changes, making edits outside the approved session) is a hard failure.

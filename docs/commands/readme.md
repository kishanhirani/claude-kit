[Documentation Index](../index.md) / commands

# Commands

This section documents every skill defined in `commands/`. Each skill is a `.md` file loaded by Claude Code as a slash command.

---

## Table of Contents

- [Overview](#overview)
- [Available Skills](#available-skills)
- [Skill File Anatomy](#skill-file-anatomy)
- [Authoring a New Skill](#authoring-a-new-skill)

---

## Overview

Skills are the core primitive of claude-kit. A skill is a markdown file that Claude Code loads and executes when a user invokes its slash command. Skills encapsulate a specific, repeatable workflow — from generating commit messages to building multi-phase implementation plans.

Each skill file lives in `commands/` and follows a strict structure:

```
commands/
├── commit.md        → /commit
├── create-docs.md   → /create-docs
├── daily-update.md  → /daily-update
├── plan.md          → /plan
└── update-docs.md   → /update-docs
```

---

## Available Skills

| Slash Command   | File                       | Purpose                                      |
|-----------------|----------------------------|----------------------------------------------|
| `/commit`       | `commands/commit.md`       | Analyse staged changes and suggest a conventional commit message |
| `/create-docs`  | `commands/create-docs.md`  | Inventory a codebase, find documentation gaps, and produce a session-driven plan for creating all missing docs |
| `/daily-update` | `commands/daily-update.md` | Generate a concise summary of today's git work |
| `/plan`         | `commands/plan.md`         | Create a structured, multi-session implementation plan |
| `/update-docs`  | `commands/update-docs.md`  | Read a documentation maintenance guide and update relevant docs based on changes made in the current session |

---

## Skill File Anatomy

Every skill file has two parts:

### 1. YAML Frontmatter

```markdown
---
description: Short description shown in the skill picker
allowed-tools: Bash(git diff --cached*), Bash(git status*)
---
```

- **`description`** — displayed in the `/` skill picker. Keep it under 80 characters.
- **`allowed-tools`** — a comma-separated list of tool permissions granted to this skill. Use the `Bash(prefix*)` pattern to allow all Bash sub-commands that start with a given prefix. Omitting this field grants no tools.

### 2. Prompt Body

The prompt body is a structured markdown document that acts as the instruction set for Claude Code when the skill is invoked. It typically contains:

- **Constraints** — hard rules (e.g., "Do NOT run `git commit`")
- **Steps** — numbered, sequential actions Claude must take
- **Output format** — the exact shape of the result Claude should produce

---

## Authoring a New Skill

1. Create a new `.md` file in `commands/` with a camelCase name (e.g., `commands/reviewPr.md`)
2. Add YAML frontmatter with `description` and `allowed-tools`
3. Write a clear prompt body: constraints first, then steps, then output format
4. Add the skill to `docs/index.md` under the `commands` section
5. Create a corresponding documentation file in `docs/commands/` (e.g., `docs/commands/reviewPrCommand.md`)
6. Update `docs/doc-maintenance/documentationProgress.md`

---

## Detailed Skill Documentation

- [commit — Commit Message Generator](./commitCommand.md)
- [create-docs — Documentation Creator](./createDocsCommand.md)
- [daily-update — Daily Work Summary](./dailyUpdateCommand.md)
- [plan — Implementation Plan Creator](./planCommand.md)
- [update-docs — Documentation Updater](./updateDocsCommand.md)

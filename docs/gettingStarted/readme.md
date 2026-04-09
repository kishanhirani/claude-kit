[Documentation Index](../index.md) / gettingStarted

# Getting Started

This section covers everything you need to install, configure, and begin using claude-kit in your projects.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuring Skills](#configuring-skills)
- [Verifying Setup](#verifying-setup)
- [Next Steps](#next-steps)

---

## Prerequisites

Before using claude-kit you need:

- **Claude Code CLI** installed and authenticated (`claude` available in your terminal)
- **Git** installed and a git repository initialised in your project
- A working **Claude Code** subscription with skill support enabled
- **code-review-graph** installed globally — required by `/plan`, `/create-docs`, and `/update-docs` for dependency analysis

Install code-review-graph:

```bash
npm install -g code-review-graph
```

After installing, build the dependency graph in your project root:

```bash
code-review-graph build
```

Run this once per project before using any skill that performs codebase analysis. Subsequent updates use:

```bash
code-review-graph update
```

> [!NOTE]
> `code-review-graph` also exposes an MCP server. Claude Code skills call it via the MCP tools (`get_impact_radius`, `query_graph`, `get_minimal_context`) — you do not need to invoke these manually. The `build` and `update` CLI commands are the only ones you need to run directly.

---

## Installation

claude-kit is a collection of skill definition files. You do not install a package — you reference or copy the `commands/` files into your Claude Code configuration.

**Option A — Clone and reference directly:**

```bash
git clone https://github.com/your-org/claude-kit
```

Point your Claude Code configuration at the `commands/` directory, or copy individual `.md` files into your project's `commands/` folder.

**Option B — Copy individual skills:**

Copy any `.md` file from `commands/` into your project's skill directory:

```bash
cp commands/commit.md your-project/.claude/commands/commit.md
cp commands/plan.md   your-project/.claude/commands/plan.md
```

---

## Configuring Skills

Each skill file in `commands/` has YAML frontmatter that controls its behaviour:

```markdown
---
description: Short description shown in the skill picker
allowed-tools: Bash(git diff --cached*), Bash(git status*)
---
```

- `description` — shown when listing available skills
- `allowed-tools` — restricts which tools Claude Code may invoke for this skill. Use `*` as a wildcard suffix to allow all sub-commands of a prefix.

> [!NOTE]
> If `allowed-tools` is omitted, Claude Code will not grant any tools to the skill automatically.

---

## Verifying Setup

After placing skill files, open a Claude Code session in your project and type `/` to confirm your skills appear in the picker:

```
/commit       — Commit Message Generator
/plan         — Implementation Plan Creator
/daily-update — Daily Work Summary
```

Run `/commit` in a repo with staged changes to confirm the skill loads and executes correctly.

---

## Next Steps

- [Commands Overview](../commands/readme.md) — detailed documentation for each skill
- [Workflows Overview](../workflows/readme.md) — end-to-end usage patterns
- [Documentation Maintenance Guide](../doc-maintenance/documentationMaintenanceGuide.md) — how to keep docs in sync with code changes

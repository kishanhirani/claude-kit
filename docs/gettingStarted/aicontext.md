[Documentation Index](../index.md) / gettingStarted / AI Context

# AI Context: Getting Started

This file provides localized context for AI assistants working on the Getting Started documentation section.

---

## 🏗 Section Identity

The **Getting Started** section covers installation, configuration, and initial verification of claude-kit. Its audience is a developer setting up claude-kit for the first time. It must be accurate, minimal, and link forward to deeper sections rather than duplicate their content.

---

## 📚 Key Terminology

- **Skill File:** A `.md` file in `commands/` loaded by Claude Code as a slash command. Has YAML frontmatter (`description`, `allowed-tools`).
- **allowed-tools:** YAML frontmatter field that gates which Bash sub-commands or tools Claude Code may invoke during skill execution.
- **Claude Code CLI:** The `claude` command-line tool by Anthropic. Users must have it installed and authenticated.
- **Skill Picker:** The in-session dropdown triggered by `/` that lists available skills.
- **code-review-graph:** A globally installed CLI (`npm install -g code-review-graph`) and MCP server used by `/plan`, `/create-docs`, and `/update-docs` to perform dependency analysis before reading source files. Must be installed and initialized (`code-review-graph build`) in each project before those skills will function correctly.

---

## 💾 Core File Affiliation

- `commands/commit.md` — commit skill
- `commands/plan.md` — plan skill
- `commands/daily-update.md` — daily update skill

---

## 🖇 Mandatory Cross-References

- `docs/gettingStarted/readme.md` — the primary getting started guide
- `docs/commands/readme.md` — skill-by-skill reference
- `docs/index.md` — master index

---

## 🎨 Formatting Rules

- Use `bash` code blocks for all terminal commands
- Use `markdown` code blocks when showing skill frontmatter examples
- Use `> [!NOTE]` callouts for configuration gotchas
- Keep steps numbered and sequential

---

## 🚨 Strict Constraints

1. Never show real API keys, tokens, or authentication credentials in examples
2. Do not document internal Claude Code implementation details — document observable behaviour only
3. Do not duplicate skill content here — link to `docs/commands/` instead

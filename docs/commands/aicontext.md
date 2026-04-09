[Documentation Index](../index.md) / commands / AI Context

# AI Context: Commands

This file provides localized context for AI assistants working on the Commands section documentation.

---

## 🏗 Section Identity

The **Commands** section documents every skill file in `commands/`. It covers skill anatomy, usage, constraints, step-by-step processes, and output formats. AI assistants editing this section must accurately reflect the actual `.md` files in `commands/` — never invent behaviour that is not in the source file.

---

## 📚 Key Terminology

- **Skill:** A `.md` file in `commands/` that Claude Code loads as a slash command. Has YAML frontmatter and a prompt body.
- **Frontmatter:** The YAML block at the top of a skill file delimited by `---`. Contains `description` and `allowed-tools`.
- **allowed-tools:** The YAML field that restricts which Bash sub-commands or Claude Code tools the skill can invoke. Pattern: `Bash(prefix*)`.
- **Prompt Body:** The instruction set below the frontmatter — constraints, steps, output format.
- **Read-only Skill:** A skill that is explicitly constrained from making any mutations (e.g., no `git commit`, no file writes). Example: `/commit`.
- **Single-Session Principle:** The core constraint of `/plan` and `/create-docs` execution — Claude executes one session at a time, never previews future sessions, and always waits for user approval before proceeding.
- **Documentation Gap:** A discrepancy between what exists in `commands/` and what is documented in `docs/` — classified as new-skill-doc, missing-section, stale-skill-doc, incomplete-skill-doc, aicontext-drift, index-drift, or changelog-missing.
- **code-review-graph:** A globally installed CLI and MCP server used by `/plan`, `/create-docs`, and `/update-docs`. Provides `get_minimal_context`, `get_impact_radius`, and `query_graph` MCP tools that identify which files are affected by a task before any source files are read. Skills call these tools — never the `detect-changes` CLI subcommand.

---

## 💾 Core File Affiliation

- `commands/commit.md` — source of truth for `/commit` behaviour
- `commands/create-docs.md` — source of truth for `/create-docs` behaviour
- `commands/plan.md` — source of truth for `/plan` behaviour
- `commands/daily-update.md` — source of truth for `/daily-update` behaviour

---

## 🖇 Mandatory Cross-References

- `docs/commands/readme.md` — commands section overview
- `docs/commands/commitCommand.md` — commit skill detail
- `docs/commands/createDocsCommand.md` — create-docs skill detail
- `docs/commands/planCommand.md` — plan skill detail
- `docs/commands/dailyUpdateCommand.md` — daily-update skill detail
- `docs/index.md` — master index

---

## 🎨 Formatting Rules

- Always use `markdown` code blocks to show frontmatter examples
- Use `bash` code blocks for terminal/git commands
- Show output format examples in plain code blocks (no language tag) to match the raw text output Claude produces
- Constraints must appear before steps in all skill documentation — mirror the source file structure

---

## 🚨 Strict Constraints

1. Documentation must accurately reflect the actual `commands/*.md` source files — read them before writing or updating documentation
2. Never document behaviour that is not in the source skill file
3. Never suggest that a read-only skill (like `/commit`) can make mutations
4. Never expose real tokens, credentials, or branch names in examples — use placeholders like `[BRANCH_NAME]`, `[REDACTED]`

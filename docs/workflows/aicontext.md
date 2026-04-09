[Documentation Index](../index.md) / workflows / AI Context

# AI Context: Workflows

This file provides localized context for AI assistants working on the Workflows section documentation.

---

## 🏗 Section Identity

The **Workflows** section documents how the skills in `commands/` are composed into repeatable development patterns. It is prescriptive — it tells developers exactly when and how to use each skill, in what order, and what to avoid. AI assistants editing this section must ensure the workflows remain accurate reflections of the actual skill behaviours documented in `docs/commands/`.

---

## 📚 Key Terminology

- **Standard Development Loop:** The canonical daily sequence: `/plan` → implement (session-by-session) → `/commit` → `/daily-update`
- **Session:** A single, bounded unit of implementation work in a plan. Must be executed one at a time with user approval between each.
- **Single-Session Principle:** The hard constraint that no plan session may be previewed, batched, or executed without prior approval.
- **Pre-Planning Questions:** The mandatory questions in Step 1 of `/plan` that must be answered before any codebase reading or plan writing begins.
- **code-review-graph:** A globally installed CLI and MCP server that builds a dependency graph of the codebase. Used in `/plan` Step 0 and `/update-docs` to identify which files are affected by a change before reading any source files. MCP tools (`get_impact_radius`, `query_graph`) are called during skills; the `update` CLI command is run after sessions to keep the graph current.

---

## 💾 Core File Affiliation

- `commands/commit.md` — source of `/commit` behaviour
- `commands/plan.md` — source of `/plan` behaviour
- `commands/daily-update.md` — source of `/daily-update` behaviour

---

## 🖇 Mandatory Cross-References

- `docs/workflows/readme.md` — workflows overview
- `docs/commands/readme.md` — skill reference
- `docs/commands/planCommand.md` — plan skill detail (especially Single-Session Principle)
- `docs/doc-maintenance/documentationMaintenanceGuide.md#code-review-graph-integration` — graph tool usage in doc workflows
- `docs/index.md` — master index

---

## 🎨 Formatting Rules

- Use numbered steps for sequential workflows
- Use tables for anti-pattern comparisons
- Use Mermaid `flowchart` if a workflow needs a visual — keep it short (under 10 nodes)
- Keep the Standard Development Loop visible early in the document — it is the most referenced pattern

---

## 🚨 Strict Constraints

1. All workflow steps must accurately reflect current skill behaviour — read `docs/commands/` before editing workflows
2. Never suggest skipping pre-planning questions or the Single-Session Principle
3. Do not invent workflow steps that reference tools or skills not documented in `commands/`
4. code-review-graph MCP tools (`get_impact_radius`, `query_graph`) must always precede source file reads in `/plan` and `/update-docs` — never document a workflow that skips this step

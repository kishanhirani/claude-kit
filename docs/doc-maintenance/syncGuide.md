[Documentation Index](../index.md) / doc-maintenance / Sync Guide

# Sync Guide

**Which documentation sections must be updated for each type of code change**

This guide is a fast lookup for determining exactly which docs need updating when a specific part of claude-kit changes. Use it before running `/commit` to ensure nothing is missed.

---

## Table of Contents

- [Skill Modified](#skill-modified)
- [Skill Added](#skill-added)
- [Skill Removed](#skill-removed)
- [Skill Renamed](#skill-renamed)
- [Workflow Pattern Changed](#workflow-pattern-changed)
- [Setup or Prerequisites Changed](#setup-or-prerequisites-changed)
- [New Terminology Introduced](#new-terminology-introduced)
- [Documentation Structure Changed](#documentation-structure-changed)
- [AI Context Drift Rules](#ai-context-drift-rules)

---

## Skill Modified

**Trigger:** Any change to an existing file in `commands/`

| Section Changed | Files to Update |
|---|---|
| `description` frontmatter | `docs/commands/readme.md` (skills table) |
| `allowed-tools` frontmatter | `docs/commands/{name}Command.md` (Allowed Tools section) |
| Step-by-step process | `docs/commands/{name}Command.md` (Step-by-Step Process section) |
| Output format | `docs/commands/{name}Command.md` (Output Format section + Examples) |
| Constraint added or removed | `docs/commands/{name}Command.md` (Constraints section) |
| Behaviour change affecting workflows | `docs/workflows/readme.md` |

**Always also update:**
- `docs/doc-maintenance/changelog.json`
- `docs/doc-maintenance/changelog.md`

---

## Skill Added

**Trigger:** New `.md` file added to `commands/`

Files to update:

1. Create `docs/commands/{skillName}Command.md` — full documentation for the new skill
2. `docs/commands/readme.md` — add row to skills table, add link under Detailed Skill Documentation
3. `docs/index.md` — add link under the `commands` section
4. `docs/doc-maintenance/documentationProgress.md` — add entry under Not Started or In Progress
5. `docs/commands/aicontext.md` — add the new file to `Core File Affiliation`
6. `docs/doc-maintenance/changelog.json` — prepend new entry
7. `docs/doc-maintenance/changelog.md` — prepend new row

---

## Skill Removed

**Trigger:** A `.md` file removed from `commands/`

Files to update:

1. Remove or archive `docs/commands/{skillName}Command.md`
2. `docs/commands/readme.md` — remove from skills table and Detailed Skill Documentation links
3. `docs/index.md` — remove the link
4. `docs/doc-maintenance/documentationProgress.md` — mark as removed
5. `docs/commands/aicontext.md` — remove from `Core File Affiliation`
6. `docs/workflows/readme.md` — remove or replace any workflow references to the skill
7. `docs/doc-maintenance/changelog.json` — prepend new entry noting removal
8. `docs/doc-maintenance/changelog.md` — prepend new row

---

## Skill Renamed

**Trigger:** A `.md` file in `commands/` is renamed

Files to update:

1. Rename `docs/commands/{oldName}Command.md` to `docs/commands/{newName}Command.md`
2. Update the breadcrumb in that file
3. `docs/commands/readme.md` — update slash command, file path, and link
4. `docs/index.md` — update the link
5. `docs/workflows/readme.md` — update all references to the old slash command
6. `docs/doc-maintenance/changelog.json` — prepend new entry
7. `docs/doc-maintenance/changelog.md` — prepend new row

---

## Workflow Pattern Changed

**Trigger:** A usage pattern, ordering, or best practice in how skills are combined changes

Files to update:

1. `docs/workflows/readme.md` — update the affected workflow section
2. If the Standard Development Loop order changed: update the loop diagram/sequence at the top of the file
3. `docs/doc-maintenance/changelog.json` — prepend new entry

---

## Setup or Prerequisites Changed

**Trigger:** Installation steps, configuration format, or prerequisites change

Files to update:

1. `docs/gettingStarted/readme.md` — update the affected section (Prerequisites, Installation, Configuring Skills, Verifying Setup)
2. `docs/doc-maintenance/changelog.json` — prepend new entry

---

## New Terminology Introduced

**Trigger:** A new concept, system name, or domain term is used that did not exist before

Files to update:

1. `docs/aicontext.md` — add a one-sentence definition to `📚 Key Terminology`
2. The relevant domain `aicontext.md` — add to its `📚 Key Terminology` section
3. `docs/doc-maintenance/changelog.json` — prepend new entry

---

## Documentation Structure Changed

**Trigger:** A folder added, removed, or renamed in `docs/`

Files to update:

1. `docs/index.md` — update all affected links and section headings
2. `docs/aicontext.md` — update `🗺 Directory Mapping`
3. All files with breadcrumbs pointing to the changed folder — update the breadcrumb paths
4. `docs/doc-maintenance/documentationProgress.md` — reflect new structure
5. `docs/doc-maintenance/changelog.json` — prepend new entry

---

## AI Context Drift Rules

> [!WARNING]
> AI context files are the most critical files in the documentation system. Stale AI context causes AI assistants to generate incorrect code and documentation. Follow these rules strictly.

| Change | Which aicontext.md to update | What to update |
|---|---|---|
| New skill added | `docs/commands/aicontext.md` | Add to `Core File Affiliation` |
| New domain folder added | `docs/aicontext.md` | Add to `🗺 Directory Mapping` |
| New terminology introduced | `docs/aicontext.md` + relevant domain `aicontext.md` | Add to `📚 Key Terminology` |
| Critical behaviour changed (e.g., Single-Session Principle) | `docs/aicontext.md` | Update `⚙️ Critical System Behaviours` |
| Skill constraint changed | `docs/commands/aicontext.md` | Update `🚨 Strict Constraints` |
| Code affiliation changed (file renamed or moved) | Relevant domain `aicontext.md` | Update `💾 Core File Affiliation` |

[Documentation Index](../index.md) / commands / update-docs

# update-docs — Documentation Updater

**Slash Command:** `/update-docs`
**Source File:** `commands/update-docs.md`
**Type:** Read and write (updates existing documentation files)

---

## Table of Contents

- [Purpose](#purpose)
- [Allowed Tools](#allowed-tools)
- [Constraints](#constraints)
- [How to Invoke](#how-to-invoke)
- [Step-by-Step Process](#step-by-step-process)
- [Example](#example)

---

## Purpose

`/update-docs` reads a documentation maintenance guide and uses the changes that occurred in the current conversation as context to update the relevant documentation files. You do not need to describe what changed — the session history carries that context. You only provide the path to the guide that defines what and how to update.

---

## Allowed Tools

```
Read
Edit
Glob
Write
```

---

## Constraints

- Only update documentation files — never modify source code or skill files
- Only update sections that are affected by the session's changes — leave unrelated sections untouched
- Follow all formatting rules defined in the maintenance guide (breadcrumbs, Mermaid diagrams, code fences, changelog format)
- Never create "v2", "notes", or "update" suffixed files — edit existing files in place
- Always update the changelog as the final step

---

## How to Invoke

```
/update-docs <path-to-maintenance-guide>
```

**Example:**

```
/update-docs docs/doc-maintenance/documentationMaintenanceGuide.md
```

The argument is the path to the maintenance guide. The skill reads that guide and determines which documentation files to update based on what happened in the current session.

---

## Step-by-Step Process

1. Read the maintenance guide at `$ARGUMENTS`
2. Review the changes made in the current conversation
3. Use the guide's Documentation File Map and update rules to identify which documentation files are affected
4. Update each affected file — only the relevant sections
5. Update the changelog as the final step

---

## Example

A session where `/plan` was modified to add a new pre-planning question:

```
/update-docs docs/doc-maintenance/documentationMaintenanceGuide.md
```

The skill reads the maintenance guide, identifies that `commands/plan.md` changed, locates `docs/commands/planCommand.md` via the file map, updates the step-by-step section to reflect the new question, and prepends a changelog entry.

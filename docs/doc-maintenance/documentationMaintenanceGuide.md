[Documentation Index](../index.md) / doc-maintenance / Documentation Maintenance Guide

# Documentation Maintenance Guide

**How to keep documentation in sync with skill and workflow changes**

This guide defines when, what, and how to update documentation after making changes to claude-kit. Use it when working with AI assistants or when manually maintaining docs.

---

## Table of Contents

- [Quick Reference](#quick-reference)
- [When to Update Documentation](#when-to-update-documentation)
- [What to Update](#what-to-update)
- [Step-by-Step Process](#step-by-step-process)
- [Changelog Entry Format](#changelog-entry-format)
- [AI Context Sync Protocol](#ai-context-sync-protocol)
- [Documentation File Map](#documentation-file-map)
- [Checklist Template](#checklist-template)

---

## Quick Reference

After any change to `commands/`, ask yourself:

1. **Does this change a skill's behaviour or output format?** → Update the skill's doc in `docs/commands/`
2. **Does this add or remove a skill?** → Update `docs/commands/readme.md` and `docs/index.md`
3. **Does this change how skills are composed?** → Update `docs/workflows/readme.md`
4. **Does this change setup or configuration?** → Update `docs/gettingStarted/readme.md`
5. **Does this introduce new terminology?** → Update `docs/aicontext.md` Key Terminology section

---

## When to Update Documentation

### Always Update For

- New skill file added to `commands/`
- Existing skill's steps, constraints, or output format changed
- Skill frontmatter (`allowed-tools`, `description`) changed
- New workflow pattern documented or existing one changed
- Setup or installation process changed
- New terminology introduced

### Consider Updating For

- Bug fixes to skill logic that change observable behaviour
- New examples added to illustrate edge cases
- Clarifications to existing constraint explanations

### Do Not Update For

- Internal prompt wording tweaks with no behavioural change
- Whitespace or formatting changes inside skill files
- Git history entries (these are in git, not docs)

---

## What to Update

### Skill Changes

**Files to update:**

- `docs/commands/{skillName}Command.md` — full skill documentation
- `docs/commands/readme.md` — update the skills table if the skill was added, removed, or renamed
- `docs/index.md` — update the commands section link if the file was renamed

**What to include:**

- Updated purpose description
- Updated allowed-tools list
- Updated step-by-step process
- Updated output format with examples
- Updated constraints

### New Skill Added

**Files to update:**

- Create `docs/commands/{skillName}Command.md`
- Add row to the skills table in `docs/commands/readme.md`
- Add link in `docs/index.md` under `commands`
- Add to `docs/doc-maintenance/documentationProgress.md`
- Update `docs/doc-maintenance/changelog.json` and `changelog.md`

### Workflow Changes

**Files to update:**

- `docs/workflows/readme.md`
- If a new workflow pattern was added, add it to the Table of Contents

### Setup / Configuration Changes

**Files to update:**

- `docs/gettingStarted/readme.md`

---

## Step-by-Step Process

### Step 1 — Identify What Changed

- Which files in `commands/` were modified?
- Was the behaviour, output format, or constraint set affected?
- Did this change affect how skills are composed (workflows)?
- Did this affect setup or getting started?

### Step 2 — Locate Relevant Documentation

Use the [Documentation File Map](#documentation-file-map) to find which doc files to update.

### Step 3 — Update the Documentation

1. Open each affected documentation file
2. Locate the relevant section
3. Update with accurate information
4. Add or update examples where relevant
5. Check all cross-reference links still point to valid files

### Step 4 — Update the Changelog

Prepend a new entry to `docs/doc-maintenance/changelog.json`:

```json
{
  "session": <next number>,
  "date": "YYYY-MM-DD",
  "focus": "Short description of what changed",
  "files": ["commands/commit.md", "docs/commands/commitCommand.md"],
  "summary": "One dense sentence covering what was built, fixed, or documented."
}
```

Also prepend a new row to `docs/doc-maintenance/changelog.md`.

### Step 5 — Sync AI Context Files

See [AI Context Sync Protocol](#ai-context-sync-protocol).

### Step 6 — Verify

- All links in updated files still resolve
- Examples are accurate and up to date
- No outdated information remains

---

## Changelog Entry Format

```json
{
  "session": 1,
  "date": "2026-03-27",
  "focus": "Initial documentation structure",
  "files": [
    "docs/index.md",
    "docs/aicontext.md",
    "docs/commands/readme.md",
    "docs/commands/commitCommand.md",
    "docs/commands/planCommand.md",
    "docs/commands/dailyUpdateCommand.md",
    "docs/gettingStarted/readme.md",
    "docs/workflows/readme.md",
    "docs/doc-maintenance/documentationMaintenanceGuide.md",
    "docs/doc-maintenance/documentationProgress.md"
  ],
  "summary": "Created initial documentation structure for claude-kit with index, global AI context, commands documentation, workflows, getting started guide, and doc-maintenance system."
}
```

---

## AI Context Sync Protocol

> [!WARNING]
> AI context files (`aicontext.md`) are the primary knowledge source for AI assistants. If they drift from reality, future AI-assisted work will produce incorrect output.

After any documentation change:

1. **Local domain context:** If you modified a specific domain (e.g., added a skill), update that domain's `aicontext.md` — ensure `Code Affiliation` and `Mandatory Cross-References` are accurate
2. **Global context:** If you added a new domain folder or changed directory structure, update the `🗺 Directory Mapping` in `docs/aicontext.md`
3. **Key Terminology:** If a new concept was introduced, add a one-sentence definition to the `📚 Key Terminology` section of `docs/aicontext.md`
4. **Critical Behaviours:** If a foundational system behaviour changed (e.g., Single-Session Principle modified), update `⚙️ Critical System Behaviours` in `docs/aicontext.md`

---

## Documentation File Map

Use this to find the right file based on what you changed:

| Code Area | Documentation File |
|---|---|
| `commands/commit.md` | `docs/commands/commitCommand.md` |
| `commands/create-docs.md` | `docs/commands/createDocsCommand.md` |
| `commands/daily-update.md` | `docs/commands/dailyUpdateCommand.md` |
| `commands/plan.md` | `docs/commands/planCommand.md` |
| `commands/update-docs.md` | `docs/commands/updateDocsCommand.md` |
| Any `commands/*.md` (new skill) | Create `docs/commands/{name}Command.md`, update `docs/commands/readme.md` and `docs/index.md` |
| Setup / installation | `docs/gettingStarted/readme.md` |
| Workflow patterns | `docs/workflows/readme.md` |
| Global terminology or architecture | `docs/aicontext.md` |

---

## Checklist Template

```markdown
## Documentation Update Checklist

**Change:** [Brief description]
**Date:** [YYYY-MM-DD]
**Files Modified:** [list]

### Updates Needed:

- [ ] Identified which docs files need updating
- [ ] Updated skill documentation (if skill changed)
- [ ] Updated commands/readme.md skills table (if skill added/removed)
- [ ] Updated docs/index.md links (if file added/removed)
- [ ] Updated docs/workflows/readme.md (if workflow pattern changed)
- [ ] Updated docs/gettingStarted/readme.md (if setup changed)
- [ ] Updated docs/aicontext.md (global terminology, directory mapping, critical behaviours)
- [ ] Updated relevant domain aicontext.md (code affiliation, cross-references)
- [ ] Prepended entry to docs/doc-maintenance/changelog.json
- [ ] Prepended row to docs/doc-maintenance/changelog.md
- [ ] Updated docs/doc-maintenance/documentationProgress.md
- [ ] Verified all cross-reference links still resolve
- [ ] Verified all examples are accurate

### Files Updated:

- [ ] `docs/[file1].md`
- [ ] `docs/[file2].md`
```

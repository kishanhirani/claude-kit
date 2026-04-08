---
description: Read a documentation maintenance guide and apply updates based on changes made in this session.
allowed-tools: Read, Edit, Glob, Write, Bash
---

# Update Documentation with Code-Review-Graph Integration

## Workflow: CODE → GRAPH → DOCS → REGENERATE

This skill performs a complete documentation update workflow that leverages code-review-graph to identify exactly which documentation needs updating (instead of guessing).

## Instructions

### Step 1: Identify Code Changes

Get the list of staged code files that were modified in this session:

```bash
git diff --cached --name-only
```

Note each modified file path (e.g., `src/services/authService.ts`, `src/routes/authRouter.ts`).

### Step 2: Analyze Dependencies with Code-Review-Graph

For each modified code file, run dependency analysis:

```bash
code-review-graph detect-changes <file-path>
```

**Parse the JSON output to identify:**
- `dependents`: Which files import/depend on this file (their docs may need updating)
- `imports`: Which files this file imports from (check if those docs are affected)
- `metrics`: Risk scores showing which dependencies are most critical

**Example:** If you modified `src/services/authService.ts`:
```bash
code-review-graph detect-changes src/services/authService.ts
```

Output shows which routes, controllers, middleware depend on authService. Document these dependencies.

### Step 3: Map Dependencies to Documentation

Using the dependency graph output, map code dependencies to documentation files:

**Mapping Rules** (from `docs/doc-maintenance/documentationMaintenanceGuide.md`):
- **Route/Router changed** → Update API reference, feature docs
- **Service changed** → Update workflow docs, business logic docs
- **Schema changed** → Update data model docs, database docs
- **Middleware changed** → Update authorization docs, security docs
- **Utility changed** → Update examples, reference docs for affected features

If Graph shows `File A depends on File B` and you changed `File B`, then documentation for `File A`'s feature may need updating.

### Step 4: Read the Documentation Maintenance Guide

```
Here is the documentation maintenance guide: $ARGUMENTS
```

Read that guide now. Focus on:
- The "Code-Review-Graph Integration" section for detailed workflows
- The "Documentation File Map" for which doc files to update
- Formatting rules for consistency

### Step 5: Update Identified Documentation Files

Based on code changes AND dependency graph analysis:

1. **Identify affected doc files** from the mapping in Step 3
2. **Update only those files** (not all docs — be precise)
3. **Update sections within those files** that correspond to the code changes:
   - API endpoints changed → update API reference section
   - Workflow changed → update flow documentation
   - Schema changed → update data models section
   - New feature → check if new doc file needed
   - Environment variables changed → update setup docs

4. **Follow formatting rules** from the guide:
   - Breadcrumbs at top of each doc
   - Code snippets in fenced blocks
   - No PII/secrets in examples
   - Mermaid diagrams for flows

### Step 6: Update Changelog

After doc updates, update both changelog files:

**`docs/doc-maintenance/changelog.json`** — prepend new entry:
```json
{
  "session": "<session-number>",
  "date": "YYYY-MM-DD",
  "focus": "Short description of changes",
  "endpoints": ["METHOD /api/path"],
  "files": ["relative/path/to/changed/file.ts"],
  "summary": "Dense sentence about what changed."
}
```

**`docs/doc-maintenance/changelog.md`** — prepend new row:
```
| <#> | YYYY-MM-DD | Focus phrase | What changed — key files or endpoints |
```

### Step 7: Regenerate Dependency Graph

After all documentation updates, regenerate the graph to keep it current for future sessions:

```bash
code-review-graph build
```

This ensures the next session's `/plan` or `/update-docs` has fresh dependency data.

### Step 8: Stage Everything and Summarize

Stage all changes (code + docs + regenerated graph):

```bash
git add -A
```

Provide a summary showing:
- Which code files changed
- Which dependency graph identified affected
- Which documentation files were updated
- Changelog entry added
- Graph regenerated

**Example Summary:**
```
Code Changes:
- src/services/authService.ts — added MFA support

Dependency Analysis (code-review-graph detect-changes):
- authController.ts (high-risk dependent)
- authRouter.ts (direct dependent)
- workspaceAuth.ts (direct dependent)

Documentation Updated:
- docs/authentication/readme.md — added MFA section
- docs/system-internals/authorization.md — updated with MFA validation
- docs/index.md — added MFA reference

Changelog:
- Added entry to changelog.json and changelog.md

Graph Regenerated:
- Ran code-review-graph build (ready for next session)

All changes staged and ready to commit.
```

---

## Key Principles

- **Precision:** Use graph to know exactly which docs to update, not guessing
- **Completeness:** Graph identifies indirect dependents (don't miss documentation)
- **Graph Freshness:** Always regenerate after updates (keeps future sessions efficient)
- **27x Context Savings:** By being precise, you avoid reading/updating unrelated documentation
- **Session Cache:** Fresh graph becomes the input for next session's /plan

---

## Integration with Documentation Guide

See `docs/doc-maintenance/documentationMaintenanceGuide.md#code-review-graph-integration` for:
- Complete workflow examples
- Dependency-to-documentation mapping strategy
- All available code-review-graph commands

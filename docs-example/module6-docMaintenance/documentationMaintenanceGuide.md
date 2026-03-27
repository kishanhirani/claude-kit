[Documentation Index](../index.md) / module6-docMaintenance / Documentation Maintenance Guide

# Documentation Maintenance Guide

This guide defines the rules for keeping Orderflow documentation in sync with the codebase. Every code change that affects documented behaviour must be followed by a documentation update before the work is considered complete.

---

## Table of Contents

- [When to Update Documentation](#when-to-update-documentation)
- [Per-Change-Type Lookup Table](#per-change-type-lookup-table)
- [Changelog Procedure](#changelog-procedure)
- [Progress Tracker Procedure](#progress-tracker-procedure)
- [Naming and Formatting Rules](#naming-and-formatting-rules)

---

## When to Update Documentation

Documentation must be updated in the same session as the code change — never deferred to a follow-up. The rule of thumb: if the change would cause a reader of the current docs to misunderstand how the system works, the docs are out of date.

Changes that always require documentation updates:

- New API endpoint added or removed
- Request or response shape changed
- New error code added
- Service method added, removed, or renamed
- Schema field added, removed, or type changed
- State machine transition added or removed
- New environment variable required
- New notification event type added

---

## Per-Change-Type Lookup Table

| Change Type | Files to Update |
|---|---|
| New API endpoint | `module2-api/readme.md` (endpoints table), new or existing endpoint file, `index.md` |
| Endpoint request/response changed | Endpoint file only |
| New error code | Endpoint file (error table), service file (error reference) |
| New service method | Service file |
| Schema field added/removed | Schema file |
| New enum value | Schema file, `aicontext.md` terminology if the enum is mentioned |
| New notification event | `notificationService.md` (supported events table) |
| New environment variable | `module1-gettingStarted/readme.md` (env config section) |
| New fulfilment partner event | `module2-api/webhooksEndpoints.md` (supported events table) |

---

## Changelog Procedure

After every documentation session, add an entry to **both** changelog files:

### changelog.json

Prepend a new object (newest first):

```json
{
  "session": <next_number>,
  "date": "YYYY-MM-DD",
  "focus": "Short phrase describing what was documented",
  "files": ["list", "of", "every", "file", "modified"],
  "summary": "One dense sentence covering what changed and why."
}
```

### changelog.md

Prepend a new table row (newest first):

```markdown
| <session> | YYYY-MM-DD | focus phrase | summary sentence |
```

---

## Progress Tracker Procedure

After every documentation session, update `documentationProgress.md`:

1. Change the status of any row that moved from incomplete to complete
2. Update the "Last Updated" date at the top
3. Update "Overall Progress" if a major section reached 100%

---

## Naming and Formatting Rules

- All folder names: **camelCase**
- All file names: **camelCase** (exception: `index.md`, `readme.md` stay lowercase)
- Every file starts with a breadcrumb: `[Documentation Index](../index.md) / module / topic`
- Monetary amounts in examples: always integers (pence/cents), never floats
- Credentials in examples: always placeholders (`[STRIPE_SECRET]`, `[JWT_TOKEN]`, `[REDACTED]`)
- Mermaid diagrams: `sequenceDiagram` for flows with external systems, `flowchart` for internal state machines

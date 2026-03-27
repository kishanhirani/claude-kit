This workflow creates a new plan folder under `plans/` in the project root with a standardized file structure and an execution guide.

**Target directory:** `plans/{kebab-case-subject}-plan/`

---

## CRITICAL: EXECUTE.md Trigger

If the user's message references, links, quotes, or opens any file named `EXECUTE.md`:

- STOP. Do NOT read `EXECUTE.md`.
- Read ONLY `03-implementation-sessions.md`.
- Find the first session that has at least one unchecked `[ ]` item.
- Present ONLY that session's diff in chat as markdown code blocks.
- Do NOT use any file modification tools until the user explicitly approves.
- Do NOT mention, preview, or acknowledge any other session.
- Do NOT summarise the overall plan or state how many sessions remain.
- End your response by asking: "Approve this session?"

To advance to the next session after approval and checklist update, the user will say one of:
- "next", "continue", "proceed", "approved", or "done"

On receiving any of those signals after a session is marked complete, read `03-implementation-sessions.md` again, find the next incomplete session, and repeat the same protocol.

---

## Step 0: Load Project Context

Before anything else, load the project's knowledge base:

1. Read `docs/aicontext.md` — global project rules, terminology, tech stack, directory mapping, and critical system behaviours.
2. Use the **Directory Mapping** in `docs/aicontext.md` to identify which domain(s) this task touches:
   - Admin config / system settings → `docs/admin-panel/`
   - Heatmaps / Rankings / CTR Booster → `docs/map-ranking/`
   - Employee Check-ins / Integrations / Widgets → `docs/map-checkin/`
   - Billing / Stripe / Subscriptions → `docs/billing/`
   - Auth / OAuth / Tokens / GBP → `docs/authentication/`
   - Posts / Moments / AI features → `docs/posts-moments/`
   - White-label / Agency / Branding → `docs/white-label/`
   - Infrastructure / DB / Cron / Logging → `docs/system-internals/`
3. If an `aicontext.md` exists inside the identified domain folder(s), read it for domain-specific context.
4. Read `docs/doc-maintenance/documentationMaintenanceGuide.md` — formatting rules, changelog procedures, and documentation file map.

This context informs every subsequent step. Use the **Key Project Terminology** section to ensure consistent language throughout the plan (e.g., "Business" = GBP entity, "Moment" = AI-assisted post, "Agency" = white-label tenant, etc.).

---

## Step 1: Ask Pre-Planning Questions

Before writing any plan file, ask ALL of the following questions and wait for complete answers. Do NOT proceed to Step 2 until they are answered.

### 1.1 — Scope of Files

- Which existing source files are in scope for this task? List every file that will be touched: schemas (`src/schema/`), services (`src/services/`), controllers (`src/controllers/`), routes (`src/routes/`), middleware (`src/middleware/`), validators, utilities.
- Are we modifying an existing feature/route, or is this entirely new functionality?
- If modifying existing: provide the exact file paths of every file that will change.

### 1.2 — Route Validators & Middleware

- Does this task require new request validators (Yup schemas — project standard)?
- Should those validators go in a **new** file, or be added to an **existing** validator file? Which one?
- Are there any new middleware or authentication guards required? New file or existing?
- Are there any new routes being added to `src/routes/apiRouter.ts` or any other router file?

### 1.3 — Logic Deep-Dive

Ask as many pointed logic questions as needed to fully understand the task. At minimum cover:

- What is the exact happy-path flow, step by step?
- What are the failure/edge cases and how should each be handled?
- What are the data dependencies — which schemas are read, written, or referenced?
- Are there race conditions or concurrency concerns?
- Does this affect any existing flows or shared services used by other features?
- Are there any external integrations (Firebase, Stripe, Google Cloud/GBP API, Facebook/Instagram Graph API) involved?
- What are the specific error messages and HTTP status codes expected for each failure case?
- Are there any email notifications involved — if so, which SMTP path (platform default vs. agency SMTP)?
- Does this touch any modularized services (e.g., `src/services/heatmapServices/`, `src/services/checkinService/`, `src/utils/emailTemplates/`)? If so, which sub-module?

### 1.4 — Documentation Impact (assess early)

- Does this change add/modify/remove API endpoints? → Will need API doc updates.
- Does this change schemas or data models? → Will need schema doc updates.
- Does this change workflows or processes? → Will need flow doc updates.
- Does this add/change environment variables? → Will need setup doc updates.
- Which `docs/` files will need updating? (Use the Documentation File Map in `docs/doc-maintenance/documentationMaintenanceGuide.md` to identify.)

---

## Step 2: Read the Codebase & Analyse Risks

Before writing any plan document, read the actual source files that are in scope. Do not write `01-current-state.md` from assumptions — read the real code first.

Key files to always check:
- `src/routes/apiRouter.ts` — to understand existing route registration and middleware guards
- Relevant schema files in `src/schema/`
- Relevant service files in `src/services/` (note: many large services have been modularized into sub-folders — check for `index.ts` barrel files)
- Relevant controller files in `src/controllers/`
- Any existing validator files that might be extended
- Existing documentation for the affected domain (from Step 0 domain mapping) — to understand what docs already say and avoid contradictions

While reading, simultaneously perform a risk analysis across all in-scope files. This analysis feeds directly into `02-problems-and-caveats.md` — it is done during the planning phase, not before execution. For every file, service, schema, and route, identify and document:

**Security vulnerabilities:**
- Any new route that could be accessed without authentication or with incorrect role guards
- Input fields that are not validated and could accept unexpected data types or sizes
- Any place where user-supplied data is passed directly to a query (injection risk)
- Exposed sensitive fields in responses that should be excluded (e.g., `select: false` fields)
- PII exposure — never include real tokens, API keys, or credentials in plan examples (use `[REDACTED_API_KEY]` or `[USER_ID]` placeholders per `docs/aicontext.md` rules)

**Data integrity risks:**
- Operations that write to the database without a transaction where multiple writes must succeed or fail together
- Cascade effects — e.g., deleting or modifying a record that is referenced by other collections
- Race conditions where two concurrent requests could produce duplicate or inconsistent records
- Missing unique indexes or constraints that the code assumes are enforced
- Fields that could be left in an inconsistent state if the service throws mid-execution

**Regression risks:**
- Existing flows that share the same service, schema, or utility being modified — what breaks if the signature or behaviour changes?
- Other routes or controllers that import and depend on any file being changed
- Any shared middleware or validation that, if modified, affects routes outside the current scope
- Check `docs/doc-maintenance/changelog.md` for recent changes to the same files — avoid conflicting with in-flight work

**Logic & business rule risks:**
- Edge cases where the happy-path assumptions do not hold (e.g., empty arrays, null references, zero-credit states)
- Conditional branches that are not covered by the implementation sessions
- Ordering dependencies — steps that must happen in a specific sequence that the plan does not enforce
- Unified Identity implications — does this affect the relationship between `User` role records, owner employee accounts, or the backfill system?

**Email & notification risks (if applicable):**
- Emails triggered by the new logic that could fire unintentionally (e.g., on re-saves, on partial updates)
- SMTP path assumptions — platform default vs. agency SMTP — that could send from the wrong sender
- Missing template variables that would cause email rendering to silently fail or produce broken output
- Fire-and-forget email patterns (detached IIFEs) — ensure they don't block API responses

All identified risks are written into `02-problems-and-caveats.md` with an agreed resolution for each (derived from the pre-planning Q&A answers in Step 1 or flagged as an open question for the user to resolve before execution begins).

---

## Step 3: Create the Plan Folder and Files

Create the following files inside `plans/{kebab-case-subject}-plan/`:

### Required Files

| File | Purpose |
|---|---|
| `01-current-state.md` | Exact snapshot of the current codebase state relevant to this task — schemas, routes, services, middleware. Must be sourced from actually reading the files in Step 2. |
| `02-problems-and-caveats.md` | All known issues, edge cases, gotchas, and constraints identified. |
| `03-implementation-sessions.md` | Full implementation plan broken into numbered sessions. Each session must be self-contained and verifiable. |
| `EXECUTE.md` | Granular execution guide — one step per atomic change, referencing exact file paths. Designed to be followed session-by-session with a stop-and-verify checkpoint after each session. |

### File Naming

- Folder: `kebab-case` matching the subject — e.g. `plans/agency-widget-email-changes-plan/`
- Ordered docs: `01-`, `02-`, `03-` prefix
- `EXECUTE.md` — always uppercase, no number prefix

### Standard File Header (every .md file)

```markdown
# {Title}

**Plan:** {Subject}
**File:** {NN-filename or EXECUTE}
**Created:** {YYYY-MM-DD}

---
```

---

## Step 4: Structure of `03-implementation-sessions.md`

Each session block must follow this structure:

```markdown
## Session N: {Session Title}

**Goal:** One sentence describing what this session achieves.
**Files changed:**
- `src/path/to/file.ts`

### Changes

1. {Specific change description}
2. {Specific change description}

### Verification

**`src/path/to/file.ts` — {function or concept being changed}**
- [ ] {Atomic check scoped to this specific change}
- [ ] {Atomic check scoped to this specific change}

**`src/path/to/other.ts` — {function or concept}**
- [ ] {Atomic check scoped to this specific change}
```

**Verification section rules:**
- Group checklist items under a bold label for each changed file or function — e.g. `**src/services/foo.ts — functionName**`.
- Each `- [ ]` item must be scoped to a single observable fact about that specific change: "X is exported", "returns Y when Z", "field A is present in schema B".
- Aim for 1–4 items per group. The session file is the **single source of truth** for checklist progress — `EXECUTE.md` contains no checklists.


---

## Step 5: Structure of `EXECUTE.md`

`EXECUTE.md` is a **protocol file only**. It must never contain code, file paths, session titles, diffs, step listings, or any plan content. All of that lives exclusively in `03-implementation-sessions.md`.

The file must be 5 lines maximum. Use this exact template — nothing else:

```markdown
# EXECUTE — {Subject}

> Read `03-implementation-sessions.md`.
> Find the first session with an unchecked `[ ]` item.
> Present only that session's diff. Wait for approval.
> After executing and updating the checklist, stop and wait.
> User says "next" to advance.
```

### Execution rules (enforced at runtime)

- One session at a time. Never present Session N+1 alongside Session N.
- Do NOT use file modification tools until the user explicitly approves.
- After approval, execute ONLY the approved session — no scope creep.
- After executing, update the checklist in `03-implementation-sessions.md`, then stop. End with: "Session N complete. Ready when you are." — nothing more.
- Do NOT mention future sessions, remaining session counts, or what comes next.

---

## Step 6: Final Session — Documentation Update

Always append a documentation update as the **last session** in `03-implementation-sessions.md` and `EXECUTE.md`.

This session is **not optional** — it ensures code changes never drift from documentation. Follow these rules from the project's documentation standards:

### 6.1 — Identify What Needs Updating

Use the assessment from Step 1.4 and the Documentation File Map in `docs/doc-maintenance/documentationMaintenanceGuide.md` to determine exactly which docs files to update. At minimum check:

- **API endpoints added/modified/removed?** → Update the domain's `apiReference.md` or equivalent, plus `docs/index.md` if a new concept or section was introduced.
- **Schema/data model changes?** → Update `dataModels.md` / `businessModel.md` or equivalent in the domain folder.
- **Workflow/process changes?** → Update flow documentation and any Mermaid diagrams.
- **Environment variable changes?** → Update `docs/getting-started/serverSetup.md`.
- **New feature?** → May need a new doc file in the appropriate `docs/` subfolder + update `docs/index.md`.
- **System config / admin setting changes?** → Update `docs/admin-panel/system-configuration.md`.

### 6.2 — Documentation Formatting Rules

All documentation changes must follow:

- Every markdown file starts with a breadcrumb: `[Documentation Index](../index.md) / [category] / [file]`
- Use Mermaid.js for sequence and flow diagrams
- API references must include: URL, Method, Auth Requirements, Request/Response samples, and common error codes
- Code snippets in fenced blocks with correct language tags (`typescript`, `json`, `bash`)
- Never expose PII, API tokens, or credentials — use placeholders
- Never create fragmented "v2" or "notes" files — update existing docs in place

### 6.3 — Update the Changelog

After documentation changes, update both:

- `docs/doc-maintenance/changelog.json` — **prepend** a new entry (newest first):
  ```json
  {
    "session": "<next number>",
    "date": "YYYY-MM-DD",
    "focus": "Short description of the session theme",
    "endpoints": ["METHOD /api/path"],
    "files": ["relative/path/to/changed/file.ts"],
    "summary": "One dense sentence covering what was built/fixed/documented."
  }
  ```
- `docs/doc-maintenance/changelog.md` — **prepend** a new row to the table (newest first):
  ```
  | <#> | YYYY-MM-DD | Focus phrase | What changed — key files or endpoints |
  ```

### 6.4 — Session Structure for Doc Update

The documentation session in `03-implementation-sessions.md` must list:
- Every `docs/` file that will be created or modified
- Specific sections within each file that change
- Changelog entry details

The corresponding `EXECUTE.md` section must have one step per doc file change, ending with the changelog update step.

---

## Conventions & Constraints (always enforced)

These apply across all plan files and execution:

- **Tech stack:** Node.js (ESM), TypeScript, Express.js 5, MongoDB/Mongoose, Yup validation, node-cron. Do not introduce patterns outside these.
- **Modular services:** Large services have been split into sub-modules (e.g., `src/services/heatmapServices/`, `src/services/checkinService/`, `src/utils/emailTemplates/`). New code should follow the existing modular pattern rather than adding to monolithic files.
- **Existing patterns first:** Always follow patterns found in the codebase during Step 2. Never deviate from the project's established conventions.
- **No fragmented docs:** Never create "v2", "notes", or "update" suffixed documentation files. Edit existing files only.
- **Consistent terminology:** Use the Key Project Terminology from `docs/aicontext.md` throughout all plan files.
- **Breadcrumbs:** Every documentation file must start with a breadcrumb link to `[Documentation Index](../index.md)`.
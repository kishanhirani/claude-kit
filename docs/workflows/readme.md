[Documentation Index](../index.md) / workflows

# Workflows

This section documents end-to-end usage patterns that combine multiple claude-kit skills. It covers recommended day-to-day flows, best practices, and efficiency guidelines.

---

## Table of Contents

- [The Zero-Hallucination Methodology](#the-zero-hallucination-methodology)
- [Why Complete Documentation Is the Prerequisite](#why-complete-documentation-is-the-prerequisite)
- [How to Start a Plan Session](#how-to-start-a-plan-session)
- [The Standard Development Loop](#the-standard-development-loop)
- [Workflow: Feature Development](#workflow-feature-development)
- [Workflow: Bug Fix](#workflow-bug-fix)
- [Workflow: Documentation Update](#workflow-documentation-update)
- [Efficiency Guidelines](#efficiency-guidelines)
- [Anti-Patterns to Avoid](#anti-patterns-to-avoid)

---

## The Zero-Hallucination Methodology

The plan workflow exists to solve one specific, recurring failure mode: an AI that has not fully understood the problem starts modifying code, makes an incorrect assumption midway through, and by the time you notice the mistake the session is deep, the context is partially compacted, and correcting course now costs more than starting over.

This is not a Claude limitation — it is a consequence of how any long, stateful session degrades:

- Every token of execution output that piles up compresses earlier context.
- Compressed context loses nuance: the constraint you mentioned at message 3 is no longer live at message 40.
- Stopping mid-execution and re-directing adds more output, compresses more context, and worsens the drift.
- The longer you stay in a broken session trying to steer it back, the more the next response hallucinates.

The plan workflow prevents this entirely by front-loading all understanding into a short, cheap, read-only session before a single file is touched.

---

## Why Complete Documentation Is the Prerequisite

The plan workflow's zero-hallucination guarantee holds **only if** the codebase has up-to-date documentation. Here is why:

When you invoke `/plan`, Claude reads `docs/aicontext.md` and the relevant domain `aicontext.md` files before reading any source code. These files define:

- The exact terminology the project uses (so the AI does not invent synonyms)
- Which directories own which responsibilities (so the AI does not place code in the wrong layer)
- Which shared services exist (so the AI does not duplicate logic or break existing callers)
- Which constraints are non-negotiable (auth guards, logging rules, error formats)

If these files are absent or stale, the AI fills the gap with assumptions. Assumptions are the source of hallucinations.

> [!WARNING]
> Run `/create-docs` before running `/plan` on any codebase that lacks `docs/aicontext.md`. A plan built without project context is a plan built on guesses.

---

## How to Start a Plan Session

This is the exact sequence for zero-hallucination implementation of any task — a new feature, a bug fix, a refactor, or an architectural change.

### Step 1 — Know your problem before opening Claude

Before typing anything, write down in plain language:

- What is broken, missing, or needs to change
- What you want the end state to look like
- Any constraints you already know (must not break X, must use Y pattern, deadline is Z)

You do not need to know the solution. If you do not know the solution, say that — the plan session will investigate and surface one. What you must know is the problem.

### Step 2 — Run `/plan`

Type `/plan` in Claude Code. Claude will:

1. Load `docs/aicontext.md` and all relevant domain context files — it is reading your project's ground truth, not guessing
2. Ask you a structured set of pre-planning questions covering scope, logic, failure cases, data dependencies, and documentation impact

**Answer every question completely.** Do not skip a question because it feels obvious. The pre-planning questions are designed to surface the edge cases and constraints that cause mid-execution derailments. A thin answer here produces a plan with unresolved assumptions.

### Step 3 — Review the plan output before approving anything

After the Q&A, Claude reads the in-scope source files, performs a risk assessment, and produces the plan folder:

```
plans/{your-task}-plan/
├── 01-current-state.md           — what exists now, in detail
├── 02-problems-and-caveats.md    — risks, edge cases, known issues
├── 03-implementation-sessions.md — all sessions with checklists
└── EXECUTE.md                    — execution guide
```

**Read all four files.** This is the review gate. At this point zero code has been written. If the plan has misunderstood the problem, you fix it here — by correcting a markdown file — at near-zero cost. The alternative is correcting a misunderstanding mid-execution inside a degraded context window.

What to check in each file:

- **`01-current-state.md`** — does the AI understand the actual current behaviour? Does it know which files matter?
- **`02-problems-and-caveats.md`** — are the real risks listed? Is the edge case you care about covered? Is there anything on this list you know is wrong?
- **`03-implementation-sessions.md`** — does each session make sense in isolation? Is there a session you would remove or split? Are the verification items observable facts (not vague goals)?
- **`EXECUTE.md`** — does the execution order make sense? Would session 2 break if session 1 had not completed?

If anything is wrong, **edit the plan files directly** and tell Claude what changed. The plan is a markdown document — you own it. Editing a plan file costs nothing. Stopping an execution session mid-way costs context, coherence, and time.

### Step 4 — Execute one session at a time

Open `EXECUTE.md` and tell Claude to start. Claude will:

1. Read `03-implementation-sessions.md` and find the first unchecked session
2. Present the proposed changes as markdown — **no files are written yet**
3. Ask for your approval

At this point you have another review gate. You can see exactly what is about to happen. If it looks wrong, say so — Claude will adjust and re-present. Nothing has been written.

Once you approve, Claude writes only the files for that session, then:

- Updates the checklist in `03-implementation-sessions.md`
- Stops and waits for you

**Do not ask Claude to do the next session in the same message.** Each session starts fresh with a read of the checklist, which means Claude always knows exactly where it is — even if you are resuming a plan from a previous conversation.

### Step 5 — Commit after each session

After each session completes, run `/commit`. This creates a clean, reviewable commit boundary. If a later session produces a bad result, you have a clear rollback point. If you commit at the end of the entire plan, you lose that granularity.

### Step 6 — Correct course without breaking the session

If, during execution, you realise a session's approach is wrong:

1. Tell Claude to stop — it will not write any more files for this session
2. Edit `02-problems-and-caveats.md` or `03-implementation-sessions.md` to reflect the corrected understanding
3. Tell Claude to re-present the current session

Because no files outside the current session have been written, and because the plan files are the ground truth, the correction is contained. You do not need to undo tool calls. You do not need to start a new conversation. The session resumes from an updated plan — not from a degraded memory of earlier messages.

This is the key advantage over ad-hoc mid-session correction: the plan is a durable, editable artifact that exists outside the context window. Correcting the plan corrects the execution path. Correcting the AI mid-message inside a long session corrects nothing durably.

### Step 7 — Resume across conversations without losing state

If a plan spans multiple days or conversations, the plan folder is your state. When you return:

1. Open a new Claude Code session
2. Reference the plan folder path
3. Tell Claude to open `EXECUTE.md`

Claude reads `03-implementation-sessions.md`, finds the first unchecked item, and picks up exactly where you left off. No re-explanation needed. No context compaction risk from the prior session. The plan is the memory.

---

## Why This Works at Any Scale

For a 2-session bug fix, the plan workflow adds one extra step (the review). For a 20-session feature, it prevents 19 opportunities for mid-execution derailment.

The token cost of the planning session is bounded and predictable — it scales with the size of the codebase read, not with the complexity of the conversation. The cost of a mid-execution correction in a long session is unbounded — it compounds with every attempt to correct.

The plan workflow shifts cost from execution-time (expensive, stateful, degrading) to planning-time (cheap, stateless, durable). That shift is why zero hallucinations is achievable: every question that could cause the AI to hallucinate at execution time has already been answered at planning time, written into a file, and reviewed by you.

---

## The Standard Development Loop

claude-kit is designed around a repeatable daily loop that keeps implementation, commits, and documentation consistently in sync:

```
/plan    →    implement (session-by-session)    →    /commit    →    /daily-update
```

1. **`/plan`** — before writing any code, create a structured plan with pre-planning questions and graph-guided codebase analysis
2. **Implement** — execute sessions one at a time via `EXECUTE.md`, waiting for approval after each
3. **`/commit`** — once a session or logical unit is complete, generate a conventional commit message
4. **`/daily-update`** — at end of day, generate a concise summary of all work completed

> [!NOTE]
> `/plan` uses the `code-review-graph` MCP tools in Step 0 to identify which files are affected by the task before reading any source code. This replaces speculative directory-wide reads with precision file selection, reducing context usage by ~27x. See [Code-Review-Graph Integration](../doc-maintenance/documentationMaintenanceGuide.md#code-review-graph-integration) for details.

---

## Workflow: Feature Development

**Trigger:** Adding a new feature, endpoint, or system capability.

### Steps

1. Open Claude Code in your project directory
2. Run `/plan` and answer all pre-planning questions thoroughly
   - Cover: scope of files, validators, logic happy path, failure cases, data dependencies, documentation impact
3. Review and approve `01-current-state.md` and `02-problems-and-caveats.md`
4. Work through `EXECUTE.md` one session at a time — approve each session before it executes
5. After each session, stage the changed files and run `/commit` to generate and apply the commit message
6. At end of day, run `/daily-update` for your standup notes
7. Ensure the final documentation session in the plan is executed to keep `docs/` in sync

### Key Rules

- Never skip the pre-planning questions in `/plan` — they surface edge cases before implementation
- Never batch multiple sessions — the Single-Session Principle exists to prevent cascading errors
- Always run `/commit` after each logical unit, not at end of day for everything at once

---

## Workflow: Bug Fix

**Trigger:** Fixing a known or reported defect.

### Steps

1. Run `/plan` with a focus on the `02-problems-and-caveats.md` — document the root cause clearly
2. Keep the plan to 1-2 sessions maximum for a targeted fix
3. Execute the fix session, verify the behaviour change
4. Run `/commit` with type `fix`
5. If the bug was in documented behaviour, update the relevant `docs/` file in the final session

### Key Rules

- Do not expand the scope of a bug fix to include unrelated improvements
- Document the root cause in `02-problems-and-caveats.md` so future context is preserved

---

## Workflow: Documentation Update

**Trigger:** Code changed without a plan, or documentation fell behind.

### Steps

1. For each modified source file, call `query_graph(pattern="importers_of", target="<file>", detail_level="minimal")` to find dependents that may also need doc updates
2. Map the modified files and their dependents to `docs/` files using the [Documentation File Map](../doc-maintenance/documentationMaintenanceGuide.md#documentation-file-map)
3. Update the affected documentation files directly — no plan needed for pure doc updates
4. Update `docs/doc-maintenance/changelog.json` and `docs/doc-maintenance/changelog.md`
5. Update `docs/doc-maintenance/documentationProgress.md` if coverage changed
6. Run `code-review-graph update` to keep the graph current
7. Run `/commit` with type `docs`

---

## Efficiency Guidelines

### Do

- Run `/plan` for any task that touches more than 2 files or involves a new pattern
- Use `allowed-tools` frontmatter to scope skills tightly — never grant more permissions than needed
- Commit after each session, not after the entire plan — smaller commits are easier to review and revert
- Keep `docs/` in sync within the same plan that produced the code change

### Avoid

- Starting implementation before `/plan` completes Step 1 (pre-planning questions)
- Asking Claude to execute multiple sessions in one message
- Using `/commit` on a dirty working tree — always stage only the files for the current logical change
- Letting `docs/` fall more than one session behind the code

---

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Correct Approach |
|---|---|---|
| "Just do it all" | Skipping `/plan` for large tasks leads to cascading mistakes and hard-to-review diffs | Always run `/plan` for multi-file changes |
| Batching sessions | Executing 3 sessions at once defeats the verification step and makes rollback difficult | One session, one approval, one commit |
| Documentation later | Docs written weeks after code changes are inaccurate and miss nuance | Execute the documentation session as the last step in every plan |
| Overwide `allowed-tools` | Granting `Bash(*)` to a read-only skill defeats the safety constraint | Scope `allowed-tools` to exactly what the skill needs |
| Reading directories speculatively | Reading entire `src/` folders to find affected files wastes context and misses transitive dependents | Use `get_impact_radius` and `query_graph` from code-review-graph before reading any source files |
| Skipping graph update | Not running `code-review-graph update` after a session leaves the graph stale for the next one | Always run `code-review-graph update` after any session that modifies files |

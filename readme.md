# claude-kit

A curated set of Claude Code skills, workflows, and documentation conventions for achieving maximum and efficient output from Claude Code.

---

## What is claude-kit?

claude-kit is a toolkit of reusable **slash commands** (skills) and **workflow patterns** that standardise how you work with Claude Code across projects. Instead of writing ad-hoc prompts, you drop a skill file into your project and invoke it with a single command.

Every skill is:

- **Constrained** — tight `allowed-tools` frontmatter prevents accidental mutations
- **Structured** — clear steps, output format, and hard constraints baked in
- **Composable** — designed to work together in a repeatable daily loop

---

## Skills

| Command | File | What it does |
|---|---|---|
| `/commit` | `commands/commit.md` | Analyses staged changes and suggests a conventional commit message. Read-only — never commits or pushes. |
| `/plan` | `commands/plan.md` | Guides Claude through pre-planning questions, codebase analysis, and risk assessment before writing code. Produces a multi-session plan folder with `EXECUTE.md`. |
| `/create-docs` | `commands/create-docs.md` | Inventories a codebase, classifies documentation gaps, and produces a session-driven plan for creating all missing docs. |
| `/daily-update` | `commands/daily-update.md` | Reads today's git log, filters cleanup noise, and outputs a concise done-list for standups or end-of-day notes. |

---

## Why `/plan` is different from just asking Claude

Most people use Claude Code by describing a task and letting it run. This works for small, self-contained changes. It breaks down for anything larger — and the breakdown is not obvious until you are already deep in a broken session.

Here is what happens without a plan:

1. You describe the task. Claude starts reading files and writing code.
2. Claude makes an assumption early on — something it inferred from the code but did not confirm with you.
3. Several files later, you notice the assumption is wrong. You tell Claude to stop and correct it.
4. Claude acknowledges the correction and continues — but the session is now long. Earlier context has been compacted. The correction does not fully propagate. The next response drifts.
5. You correct it again. The session gets longer. The drift gets worse.
6. Eventually you are stuck: you cannot move forward in this session, and you cannot easily carry the context forward to a new one.

This is not a Claude limitation. It is what happens to any stateful system when corrections compound inside a degrading context window. The longer you try to steer a broken session, the more it costs — in tokens, in time, and in the quality of the output.

**`/plan` solves this by front-loading all understanding before a single file is touched.**

### How it works

```
/plan  →  review plan  →  implement (one session at a time)  →  /commit  →  /daily-update
```

**Step 1 — `/plan` asks you everything first**

Before reading any source code, Claude asks a structured set of pre-planning questions: which files are in scope, what the happy path looks like, what the failure cases are, what data dependencies exist, what documentation needs updating. You answer completely. Thin answers produce plans with unresolved assumptions — those assumptions surface as mistakes at execution time.

**Step 2 — Claude reads the codebase and produces a plan folder**

After the Q&A, Claude reads every in-scope file, performs a risk assessment, and produces:

```
plans/{your-task}-plan/
├── 01-current-state.md           — what exists now, in detail
├── 02-problems-and-caveats.md    — risks, edge cases, known issues
├── 03-implementation-sessions.md — every change broken into sessions with checklists
└── EXECUTE.md                    — the execution guide
```

Zero code has been written at this point.

**Step 3 — You review the plan before approving anything**

Read all four files. This is the review gate. If Claude has misunderstood the problem, you fix it here — by editing a markdown file — at near-zero cost. The same misunderstanding caught at session 8 of 10 costs the entire session, the coherence of the conversation, and a rollback.

What to check:

- **`01-current-state.md`** — does Claude understand the actual current behaviour and which files matter?
- **`02-problems-and-caveats.md`** — are the real risks listed? Is your edge case covered? Is anything here wrong?
- **`03-implementation-sessions.md`** — does each session make sense in isolation? Are verification items observable facts?
- **`EXECUTE.md`** — does the execution order make sense? Would session 2 break if session 1 had not completed?

If anything is wrong, edit the plan file and tell Claude what changed. The plan is a markdown document — you own it.

**Step 4 — Execute one session at a time**

Open `EXECUTE.md` and tell Claude to start. Claude finds the first unchecked session, presents the proposed changes as markdown — no files written yet — and asks for approval. You review. If it looks wrong, say so. Nothing has been written.

Once you approve, Claude writes only that session's files, updates the checklist in `03-implementation-sessions.md`, and stops. It does not start the next session until you say so.

This is the Single-Session Principle: one session, one approval, one commit. No batching.

**Step 5 — Correct course without breaking the session**

If during execution you realise a session's approach is wrong:

1. Tell Claude to stop
2. Edit `02-problems-and-caveats.md` or `03-implementation-sessions.md` to reflect the corrected understanding
3. Tell Claude to re-present the current session

Because no files outside the current session have been written, and because the plan files are the ground truth, the correction is contained. You do not need to undo tool calls. You do not need to start a new conversation. This is the key difference from ad-hoc correction: the plan is a durable, editable artifact that exists outside the context window. Correcting the plan corrects the execution path. Correcting Claude mid-message inside a long session corrects nothing durably.

**Step 6 — Resume across conversations without losing state**

If a plan spans multiple days, the plan folder is your state. Open a new session, reference the plan folder, tell Claude to open `EXECUTE.md`. Claude reads `03-implementation-sessions.md`, finds the first unchecked item, and picks up exactly where you left off. No re-explanation. No context compaction risk from the prior session. The plan is the memory.

### Why this works at any scale

For a 2-session bug fix, `/plan` adds one extra step. For a 20-session feature, it prevents 19 opportunities for mid-execution derailment.

The token cost of the planning session scales with the size of the codebase read — bounded and predictable. The cost of a mid-execution correction in a long session is unbounded — it compounds with every attempt. The plan workflow shifts cost from execution-time (expensive, stateful, degrading) to planning-time (cheap, stateless, durable).

### The prerequisite: complete documentation

The zero-hallucination guarantee holds only if your codebase has up-to-date documentation. When Claude runs `/plan`, it reads your `docs/aicontext.md` and domain-specific context files before touching any source code. These files tell Claude the exact terminology your project uses, which directories own which responsibilities, which shared services exist, and which constraints are non-negotiable.

Without these files, Claude fills the gaps with assumptions. Assumptions are the source of hallucinations.

**Run `/create-docs` before running `/plan` on any codebase that lacks an `aicontext.md`.** See `docs-example/` in this repo for a full-scale example of what complete documentation looks like — 6 modules, full API reference, service docs, schemas, workflows, and a 6-session changelog.

---

## The Standard Development Loop

```
/create-docs  →  /plan  →  implement (one session at a time)  →  /commit  →  /daily-update
```

1. **`/create-docs`** — if your project has no `docs/aicontext.md`, run this first to build the documentation foundation that makes `/plan` reliable
2. **`/plan`** — answer pre-planning questions, let Claude read the codebase, review the risk analysis, then approve the plan
3. **Implement** — work through `EXECUTE.md` one session at a time, approving each before it executes
4. **`/commit`** — stage your changes and generate a conventional commit message
5. **`/daily-update`** — at end of day, get a clean summary of everything shipped

---

## Installation

Skills are markdown files. Drop them into your project's Claude Code commands directory or reference them directly:

```bash
# Clone the repo
git clone https://github.com/your-org/claude-kit

# Copy individual skills into your project
cp commands/commit.md       your-project/.claude/commands/commit.md
cp commands/plan.md         your-project/.claude/commands/plan.md
cp commands/create-docs.md  your-project/.claude/commands/create-docs.md
```

Then open a Claude Code session and type `/` to confirm the skills appear in the picker.

---

## Documentation

`docs-example/` contains a full-scale example documentation set for a fictional Node.js/Express project called Orderflow. It demonstrates exactly what complete, AI-ready documentation looks like across 6 modules — use it as a reference when building documentation for your own project with `/create-docs`.

| Folder | What it shows |
|---|---|
| `docs-example/module1-gettingStarted/` | Installation guide, environment config, verification steps |
| `docs-example/module2-api/` | Full REST API reference — endpoints, request/response shapes, all error codes |
| `docs-example/module3-services/` | Service layer docs with architecture diagrams, method signatures, error references |
| `docs-example/module4-dataModels/` | MongoDB schema docs — fields, enums, indexes, constraints |
| `docs-example/module5-workflows/` | End-to-end flow diagrams covering happy path, failure, cancellation, refund |
| `docs-example/module6-docMaintenance/` | Maintenance guide, progress tracker, and 6-session changelog (json + md) |

---

## Roadmap

See [roadmap.md](roadmap.md) for planned skills and future direction.

---

## Contributing

1. Fork the repo
2. Add your skill to `commands/` following the [skill authoring guide](docs/commands/readme.md#authoring-a-new-skill)
3. Add documentation in `docs/commands/`
4. Update `docs/index.md` and `docs/doc-maintenance/documentationProgress.md`
5. Open a pull request

---

## License

MIT

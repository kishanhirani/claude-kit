[Documentation Index](../index.md) / commands / commit

# commit — Commit Message Generator

**Slash Command:** `/commit`
**Source File:** `commands/commit.md`
**Type:** Read-only (no mutations)

---

## Table of Contents

- [Purpose](#purpose)
- [Allowed Tools](#allowed-tools)
- [Constraints](#constraints)
- [Step-by-Step Process](#step-by-step-process)
- [Output Format](#output-format)
- [Conventional Commit Types](#conventional-commit-types)
- [Examples](#examples)

---

## Purpose

`/commit` analyses all currently staged git changes and suggests a conventional commit message. It is a read-only skill — it never stages, commits, or pushes anything. The developer reviews the suggestion and runs `git commit` manually.

---

## Allowed Tools

```
Bash(git diff --cached*)
Bash(git status*)
Bash(git diff --staged*)
```

Only read-only git inspection commands are permitted.

---

## Constraints

- **Do NOT** run `git commit`, `git push`, or any command that mutates the repository
- **Do NOT** stage or unstage any files
- Read-only analysis only

---

## Step-by-Step Process

1. Run `git status --short` — get a summary of what is currently staged
2. Run `git diff --cached --stat` — see which files changed and by how much
3. Run `git diff --cached` — read the full staged diff
4. Analyse the diff and determine:
   - The **type** of change (`feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `style`, `perf`, `ci`, `build`)
   - The **scope** (optional — the module, file group, or domain affected)
   - A concise **subject line** (≤ 72 chars, imperative mood, no trailing period)
   - Up to 3 bullet points for the **body** if the change is non-trivial
   - Any **breaking changes** (`BREAKING CHANGE:` footer if applicable)

---

## Output Format

```
<type>(<scope>): <subject>

- <what changed and why, bullet 1>
- <bullet 2 if needed>
- <bullet 3 if needed>

BREAKING CHANGE: <description>   ← only if applicable
```

Followed by a one-line explanation of why that type and scope were chosen.

> [!NOTE]
> If there are no staged changes, `/commit` says so clearly and stops — it does not produce a message for an empty diff.

---

## Conventional Commit Types

| Type       | When to Use                                              |
|------------|----------------------------------------------------------|
| `feat`     | New feature or behaviour added                           |
| `fix`      | Bug fix                                                  |
| `refactor` | Code change that neither adds a feature nor fixes a bug  |
| `chore`    | Build, tooling, dependency, or config changes            |
| `docs`     | Documentation only changes                              |
| `test`     | Adding or modifying tests                               |
| `style`    | Formatting, whitespace — no logic change                |
| `perf`     | Performance improvement                                  |
| `ci`       | CI/CD pipeline changes                                   |
| `build`    | Build system changes                                     |

---

## Examples

### Example — New Feature

Staged: a new API endpoint in `routes/heatmapRouter.ts` and a service function in `services/heatmapServices/createHeatmap.ts`.

```
feat(heatmaps): add createHeatmap endpoint with credit deduction

- POST /api/heatmap accepts keyword list and grid radius
- Deducts credits per pin before inserting HeatMap document
- Returns 402 when user has insufficient credits
```

Reasoning: `feat` because new functionality was added; `heatmaps` scope because all changes are within the heatmap domain.

---

### Example — Bug Fix

Staged: fix to `middleware/authMiddleware.ts` correcting a JWT expiry check.

```
fix(auth): correct JWT expiry comparison to use seconds not milliseconds
```

Reasoning: `fix` because this corrects broken behaviour; `auth` scope because the change is in authentication middleware.

---

### Example — Documentation

Staged: updates to `docs/commands/commitCommand.md`.

```
docs(commands): document commit skill output format and examples
```

Reasoning: `docs` because only documentation changed; `commands` scope because the affected file is under the commands section.

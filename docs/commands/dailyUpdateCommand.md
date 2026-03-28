[Documentation Index](../index.md) / commands / daily-update

# daily-update — Daily Work Summary

**Slash Command:** `/daily-update`
**Source File:** `commands/daily-update.md`
**Type:** Read-only (git log inspection only)

---

## Table of Contents

- [Purpose](#purpose)
- [Allowed Tools](#allowed-tools)
- [Step-by-Step Process](#step-by-step-process)
- [Filtering Rules](#filtering-rules)
- [Output Format](#output-format)
- [Example Output](#example-output)

---

## Purpose

`/daily-update` reads the git log for today's commits, inspects the actual code changes behind each commit, filters out noise (formatting, cleanup, comments), and produces a concise done-list of meaningful functional changes. It is designed for end-of-day reporting and standup preparation.

---

## Allowed Tools

`/daily-update` uses read-only git commands:

```bash
git log --since="midnight" --pretty=format:"%h - %s (%an, %ar)" --no-merges
git show <hash> --stat
git show <hash>
```

---

## Step-by-Step Process

1. Run `git log --since="midnight"` to get all commits made since midnight today
2. For each commit hash, run `git show <hash> --stat` to see which files changed
3. Run `git show <hash>` for the full diff of each commit
4. Filter out cleanup tasks (see [Filtering Rules](#filtering-rules))
5. Group related commits from different hashes into a single logical item where appropriate
6. Output the final done-list

---

## Filtering Rules

### Exclude (do not surface in output)

- Console log additions or removals
- Unused variable cleanup
- Comment additions or removals
- Code formatting or whitespace changes
- Lint or style fixes with no behaviour change

### Include (always surface in output)

- New features or endpoints added
- Bug fixes with observable behaviour change
- Refactors that change module structure or call patterns
- Configuration or environment variable changes
- Dependency additions or removals
- Any change to `commands/`, `docs/`, or CI/CD files

---

## Output Format

Each item on its own line, no blank lines between items, no closing summary:

```
- [Description of functional change]. -Done.
- [Description of functional change]. -Done.
- [Description of functional change]. -Done.
```

### Format Rules

- Start each line with `- `
- Use past-tense verbs: `Added`, `Implemented`, `Fixed`, `Refactored`, `Enhanced`, `Removed`
- Each item clearly describes one functional change
- Every line ends with `. -Done.` — period, space, hyphen, `Done`, period
- No blank lines between items
- No final summary statement or closing line

> [!WARNING]
> If there are no commits since midnight, `/daily-update` says so clearly and stops — it does not produce a fabricated list.

---

## Example Output

```
- Added POST /api/heatmap endpoint with grid radius and keyword list support. -Done.
- Fixed JWT expiry comparison in authMiddleware to use seconds instead of milliseconds. -Done.
- Refactored heatmapServices into domain-scoped sub-modules under src/services/heatmapServices/. -Done.
- Removed deprecated /api/v1/legacy-scan route and its controller. -Done.
```

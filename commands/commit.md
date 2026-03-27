---
description: Analyse staged files and suggest a conventional commit message. Does NOT commit or push anything.
allowed-tools: Bash(git diff --cached*), Bash(git status*), Bash(git diff --staged*)
---

# Commit Message Generator

**Important constraints — follow these strictly:**
- Do NOT run `git commit`, `git push`, or any command that mutates the repo.
- Do NOT stage or unstage any files.
- Read-only analysis only.

## Steps

1. Run `git status --short` to get a summary of what is staged.
2. Run `git diff --cached --stat` to see which files changed and how much.
3. Run `git diff --cached` to read the full staged diff.
4. Analyse the diff and determine:
   - The **type** of change (feat / fix / refactor / chore / docs / test / style / perf / ci / build)
   - The **scope** (optional — the module, file group, or domain affected)
   - A concise **subject line** (≤ 72 chars, imperative mood, no trailing period)
   - Up to 3 bullet points for the **body** if the change is non-trivial
   - Any **breaking changes** (`BREAKING CHANGE:` footer if applicable)

## Output format

Present the result exactly like this:

```
<type>(<scope>): <subject>

- <what changed and why, bullet 1>
- <bullet 2 if needed>
- <bullet 3 if needed>

BREAKING CHANGE: <description>   ← only if applicable
```

Then add a short one-line explanation of **why you chose that type and scope**, so the developer can sanity-check your reasoning.

If there are no staged changes, say so clearly and stop.

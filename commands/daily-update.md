---
description: daily update
---

# Daily Update Summary Workflow

## Objective
Generate a daily update summary for all work done on the project today.

## Instructions

Follow these rules strictly:

### 1. Check Git Log for Today's Commits
```bash
git log --since="midnight" --pretty=format:"%h - %s (%an, %ar)" --no-merges
```

### 2. Examine Actual Code Changes
- For each commit, look at the actual code changes (diffs/stats)
- Verify what was actually implemented rather than just trusting the commit message
- Use commands like:
  ```bash
  git show <commit-hash> --stat
  git show <commit-hash>
  ```

### 3. Filter Out Cleanup Tasks
**Exclude strictly "cleanup" tasks:**
- Removing console logs
- Deleting unused scripts
- Minor comment fixes
- Formatting changes only
- Whitespace adjustments

**Focus only on:**
- Functional changes
- New features
- Refactors
- Bug fixes
- Configuration updates
- Dependency changes

### 4. Combine Related Commits
- If multiple commits address the same task or feature, combine them into a single update item
- Group by logical functionality rather than individual commits

### 5. Format Output

**Use this exact format:**

```
- [Description of the actual change made]. -Done.
- [Description of the actual change made]. -Pending.
- [Description of the actual change made]. -In Progress.
- [Description of the actual change made]. -Blocked.
- [Description of the actual change made]. -Cancelled.
```

**Important formatting rules:**
- Each update item is on its own line
- Format: `- [Description]. -[Status].`
- Note: There is a space before the status tag
- Each description ends with a period before the status tag
- No blank lines between items
- No final summary or closing statement

**Available statuses:**
- `-Done.` — change is complete and committed
- `-Pending.` — change is planned or waiting to start
- `-In Progress.` — change is partially implemented
- `-Blocked.` — change is blocked by a dependency or issue
- `-Cancelled.` — change was scoped but dropped

**Example output:**
```
- Added optional email field to user profile with format validation. -Done.
- Added role-based access control to the admin dashboard routes. -Done.
- Implemented scheduled job to process pending queue items every 5 minutes. -Done.
- Made phone number mandatory for users during profile creation. -Done.
- Enhanced search to filter results by user status. -Done.
- Moved metadata field to top level in event payload structure. -Done.
- Implemented payment integration with order creation and callback handling. -Done.
- Implemented assisted booking flow with assignment, validation, and notifications. -Done.
- Refactored service routes with role-based access control middleware. -Done.
- Fixed optional field handling in notification payload. -Done.
- Restructured documentation into organised section-based folders with topic files. -Done.
- Migrate legacy auth tokens to new session model. -Pending.
- Add retry logic to outbound webhook dispatcher. -In Progress.
- Resolve rate-limiting issue on third-party API integration. -Blocked.
```

## Important Notes
- Be specific about what was changed, not just what the commit message says
- Focus on the "what" and "why" of changes
- Keep descriptions concise but informative (one line per item)
- Use past tense verbs (Added, Implemented, Fixed, Refactored, Enhanced, etc.)
- Each item should clearly describe a functional change
- Every line must end with `. -[Status].` where status is one of: Done, Pending, In Progress, Blocked, Cancelled
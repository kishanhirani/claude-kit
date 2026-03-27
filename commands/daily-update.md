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
- [Description of the actual change made]. -Done.
- [Description of the actual change made]. -Done.
```

**Important formatting rules:**
- Each update item is on its own line
- Format: `- [Description]. -Done.`
- Note: There is a space before `-Done.`
- Each description ends with a period before `-Done.`
- No blank lines between items
- No final summary or closing statement

**Example output:**
```
- Added optional PAN field to bank details with format validation. -Done.
- Added 'doctor' and 'therapist' occupations and optional registration_number to user profiles. -Done.
- Implemented payment reconciliation cron job to auto-capture pending Razorpay payments every 5 minutes. -Done.
- Made registration_number mandatory for doctors during profile creation/editing. -Done.
- Enhanced admin user search to include registration_number for doctor searches. -Done.
- Moved role field to top level in referral data structure (from referred object to referrer). -Done.
- Implemented provider joining fee payment system with Razorpay integration. -Done.
- Implemented doctor-assisted booking flow with user assignment, validation, and notifications. -Done.
- Refactored service routes to doctorRouter with role-based access control. -Done.
- Fixed optional order_id handling in booking notification payload. -Done.
- Restructured documentation into organized section-based folders with topic files and READMEs. -Done.
```

## Important Notes
- Be specific about what was changed, not just what the commit message says
- Focus on the "what" and "why" of changes
- Keep descriptions concise but informative (one line per item)
- Use past tense verbs (Added, Implemented, Fixed, Refactored, Enhanced, etc.)
- Each item should clearly describe a functional change
- Every line must end with `. -Done.` (period, space, hyphen, Done, period)
# Roadmap

---

## v0.1 — Foundation (current)

- [x] `/commit` — conventional commit message generator
- [x] `/plan` — multi-session implementation plan creator with Single-Session Principle enforcement
- [x] `/daily-update` — daily work summary from git log
- [x] Full documentation structure (`docs/`) with index, aicontext, sync guide, progress tracker

---

## v0.2 — Review & Quality

- [ ] `/review-pr` — review a pull request diff and produce structured feedback (severity-labelled, actionable)
- [ ] `/test-plan` — given a feature description or diff, generate a test checklist covering happy path, edge cases, and failure modes
- [ ] `/changelog` — generate a user-facing changelog entry from recent commits

---

## v0.3 — Context & Knowledge

- [ ] `/summarise-pr` — produce a concise PR description from branch diff and commit messages
- [ ] `/onboard` — read a codebase and produce a structured onboarding briefing (architecture, key files, gotchas)
- [ ] `/debug` — structured debugging workflow: reproduce → isolate → hypothesise → verify

---

## v0.4 — Documentation Automation

- [ ] `/doc-update` — given a list of changed files, identify which docs are stale and produce update diffs
- [ ] `/aicontext-sync` — audit all `aicontext.md` files against current source files and flag drift

---

## Future

- Skill composition — chain multiple skills in a single invocation
- Project-specific skill overrides — local `commands/` takes precedence over global kit
- Skill registry — browseable index of community-contributed skills

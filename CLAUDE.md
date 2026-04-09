# Project Rules

## MCP Servers
- Context7 MCP is available: always use it to fetch up-to-date docs for Node.js, Express, Mongoose, and any npm package before generating code.
- Filesystem MCP is available: read actual project files before proposing changes — never assume file structure or contents.
- Sequential Thinking MCP is available: always use it for multi-step tasks, architecture decisions, debugging complex issues, and anything requiring a plan before implementation.

## CLI Tools & Utilities
- **code-review-graph** (globally installed): Use to analyze code dependencies and reduce context usage by 27x on medium-large codebases.
  
  **Core Commands:**
  - `code-review-graph build` — Parse entire codebase and generate dependency graph
  - `code-review-graph update` — Incremental update for changed files only (faster)
  - `code-review-graph detect-changes` — Risk-scored change impact analysis (PRIMARY FOR /plan)
  - `code-review-graph status` — View graph statistics and coverage
  - `code-review-graph watch` — Auto-update graph on file changes
  - `code-review-graph visualize` — Generate interactive HTML visualization
  
  **Utility Commands:**
  - `code-review-graph install` — Auto-detect and configure for your platform
  - `code-review-graph wiki` — Generate markdown wiki from code communities
  - `code-review-graph eval` — Run evaluation benchmarks
  - `code-review-graph serve` — Start MCP server for integration
  
  **Multi-Repo Registry:**
  - `code-review-graph register <path>` — Register repo in multi-repo registry
  - `code-review-graph unregister <id>` — Remove repo from registry
  - `code-review-graph repos` — List registered repositories
  
  **Primary Usage in Workflows:**
  - **In /plan:** Use the `code-review-graph` MCP tools — call `get_impact_radius` with `changed_files=[<entry-point>]` for blast radius, and `query_graph(pattern="importers_of", target=<file>)` for dependency lookup. Never use the `detect-changes` CLI for this — it only reads git-staged diffs, not explicit files.
  - **In /update-docs:** Use `query_graph(pattern="importers_of")` on each modified file to identify affected dependents, then map those to docs. After all updates, run `code-review-graph update` to keep the graph fresh.
  - **Context Optimization:** Always call `get_minimal_context(task="<task>")` first. Use `detail_level="minimal"` on all MCP tool calls. Only escalate to "standard" when minimal output is insufficient.
  - **detect-changes CLI:** Only valid in git hooks (PreCommit) — reads staged diffs. Do not use it to analyze specific files.

## Workflow
- Never execute code, file modifications, or terminal commands without prior authorization.
- Always present a plan or diff before implementing anything.
- Wait for explicit approval before making changes.
- Act as a pure logic provider — propose, then wait for user verification and test results.
- Never generate README, guide, or how-to documents unless explicitly requested.
- Never run, compile, or verify code in your own environment. User handles all testing.

## Code Style
- No emojis in code or comments.
- No multiline comments at the top of functions.
- Keep comments minimal and inline only where necessary.
- Write self-documenting code: meaningful names, small functions, clear structure.

## Principles
- Follow SOLID principles.
- DRY: extract common logic into reusable functions/modules.
- KISS: prefer simple design over clever complexity.

## Logging & Error Handling
- Use low-cardinality log messages: `logger.info({ id, foo }, 'Message')`.
- Never log sensitive data (passwords, tokens, PII).
- Use appropriate log levels: debug, info, warn, error.
- Implement robust error handling throughout.

## Stack Defaults
- Backend: Node.js, Express, MongoDB (Mongoose)
- Mobile: React Native, Expo, gluestack-ui v2, NativeWind
- Language: TypeScript unless project is already in JS


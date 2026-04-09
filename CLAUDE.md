# Project Rules

## MCP & Workspace Constraints
- **Filesystem MCP**: OUTSIDE workspace ONLY (e.g., `~/.config`). 
- **Native Tools**: INSIDE workspace, strictly use native Read/Edit/Write/Glob/Grep for proper diffs and sandboxing.
- **Context7**: Fetch latest docs (Node, Express, Mongoose, npm) prior to code generation.
- **Sequential Thinking**: Mandatory for multi-step tasks, architecture decisions, and complex debugging.

## code-review-graph Workflow
- **Context Optimization**: Call `get_minimal_context(task="<task>")` first. Use `detail_level="minimal"` on all MCP tool calls. Escalate to "standard" only if minimal fails.
- **Planning (`/plan`)**: Call MCP `get_impact_radius(changed_files=[<entry>])` and `query_graph(pattern="importers_of", target=<file>)`. NEVER use `detect-changes` CLI here.
- **Documentation (`/update-docs`)**: Use MCP `query_graph(pattern="importers_of")` on modified files. After updates, run CLI `code-review-graph update`.
- **Pre-Commit**: CLI `detect-changes` is strictly for git hooks (reads staged diffs). 

## AI Execution Bounds
- **Authorization**: NEVER execute code, run commands, or modify files autonomously. 
- **Process**: Propose plan/diff -> Await explicit user approval -> User handles all testing.
- **Output Limits**: NO generating READMEs or guides unless explicitly requested.

## Stack & Code Standards
- **Core Stack**: TypeScript, Node.js, Express, Mongoose, React Native, Expo, gluestack-ui v2, NativeWind.
- **Style**: SOLID, DRY, KISS. Self-documenting code. NO emojis. NO top-level multiline comments. Minimal inline comments.
- **Logging**: Robust error handling. Low-cardinality (`logger.info({ id }, 'Msg')`). Standard levels. ZERO sensitive data/PII.
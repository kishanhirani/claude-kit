# Project Rules

## MCP Servers
- Context7 MCP is available: always use it to fetch up-to-date docs for Node.js, Express, Mongoose, and any npm package before generating code.
- Filesystem MCP is available: read actual project files before proposing changes — never assume file structure or contents.
- Sequential Thinking MCP is available: always use it for multi-step tasks, architecture decisions, debugging complex issues, and anything requiring a plan before implementation.

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


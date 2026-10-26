# Shared project context

Keep only concise, verified facts that are relevant across the repositories in this workspace. Add this folder to the multi-root `.code-workspace` file and explicitly direct agents to read relevant documents for cross-project tasks.

Suggested documents:

- `architecture-overview.md`: system boundaries and ownership.
- `dependency-map.md`: allowed dependency direction and integration points.
- `cross-project-contracts.md`: current APIs, events, and schemas, with sources.
- `glossary.md`: shared domain terms.
- `decisions/`: accepted cross-project decisions.

Link claims to source code or approved documentation. Avoid credentials, private keys, customer data, and repository-specific rules better kept in that repository's `AGENTS.md` or `.github/copilot-instructions.md`.

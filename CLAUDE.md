# Webapp Toolkit — Agent Delegation

This project includes specialist agents. **Always delegate to these agents rather than handling their domains inline:**

| When the task involves… | Agent |
|---|---|
| Architecture decisions, system design, technology choices, technical planning, or performance strategy | `tech-lead-architect` |
| Writing, refactoring, reviewing, or testing TypeScript code | `typescript-code-writer` |
| UI components, page layouts, CSS, semantic HTML, or accessibility | `ux-design-accessibility` |
| SvelteKit routes, Svelte 5 components, form actions, load functions, or API routes | `sveltekit-developer` |

When a request clearly falls within one of these domains, use the Agent tool to delegate immediately — do not attempt the work inline.

**Never spawn multiple agents in parallel.** Always run one agent at a time and wait for it to complete before starting the next. This allows the user to review and verify each change before proceeding.

## Verification

Before reporting a change as complete, verify it is acceptable by running tests, static analysis checks, and the build. Read `package.json` to find the exact script commands — do not guess or hardcode them.

1. Read `package.json` and identify the scripts for: tests, linting/type-checking, and build.
2. Run the test script.
3. Run the static analysis script (lint, type-check, or equivalent).
4. Run the build script.

All three must pass without errors before the task is considered done.

## Coding Standards

All code in this project follows these control flow principles regardless of agent:

- **Prefer early returns and guard clauses** over `if/else` and `else if` chains.
- Never write an `else` branch after a `return`, `throw`, or `continue` — it adds nesting for no benefit.
- Validate preconditions at the top of a function; keep the happy path as the unindented final statement.
- **No magic numbers or magic strings** — all literals must be extracted into named constants.

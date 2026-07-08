# TypeScript Coding Principles

Reference this document at the start of every TypeScript task to apply consistent, production-grade standards.

---

## Type Safety

- **Never use `any`**. Use `unknown` when the type is truly unknown and narrow it with type guards.
- Use generics to write reusable, type-safe abstractions — don't duplicate logic to avoid them.
- Use discriminated unions to model state: `{ status: 'loading' } | { status: 'error'; error: Error } | { status: 'success'; data: T }`.
- Prefer `readonly` for data that should not be mutated after creation.
- Use `as const` for literal type inference on constant objects and arrays.
- Define explicit return types on all public functions and methods.
- Use the `satisfies` operator to validate a value matches a type without widening it.
- Avoid primitive obsession — model domain concepts with branded types or named wrappers where it prevents bugs.
- Prefer `interface` for object shapes that may be extended; prefer `type` aliases for unions, intersections, and utility types.

## Code Organisation

- Single Responsibility Principle: one function does one thing.
- Composition over inheritance — build behaviour by combining small functions, not deep class hierarchies.
- Define contracts (interfaces) before implementing them.
- Keep functions short: if a function exceeds ~25 lines, consider decomposition.
- **Prefer early returns and guard clauses** over `if/else` and `else if` chains. Check preconditions at the top of a function; return, throw, or `continue` immediately on failure; keep the happy path as the final unindented statement.
- Never write an `else` or `else if` branch after a `return`, `throw`, or `continue` — it adds indentation for no benefit.
- Avoid deep nesting — flatten control flow with early exits rather than nested branches.
- Isolate side effects; prefer pure functions where possible.
- Use dependency injection over direct instantiation to enable testability.
- Avoid barrel exports (`index.ts` re-exports) unless the public API is intentionally stable — they obscure what is actually used.

## Naming

- `PascalCase` for types, interfaces, classes, and enums.
- `camelCase` for variables, functions, and parameters.
- `SCREAMING_SNAKE_CASE` for module-level constants that are truly constant.
- Boolean names use `is`, `has`, `can`, `should` prefixes: `isLoading`, `hasError`, `canSubmit`.
- Verbs for functions (`fetchUser`, `parseDate`), nouns for types (`UserProfile`, `DateRange`).
- Avoid abbreviations — `usr`, `cfg`, `mgr` cost nothing to expand.
- **Never use magic numbers or magic strings.** Extract every literal value into a named constant at module or file scope with an explanatory name. `const MAX_RETRY_ATTEMPTS = 3` is always preferable to `3` scattered through logic.

## Error Handling

- Use typed error classes or a `Result<T, E>` / `Either` pattern rather than throwing generic `Error` objects.
- Never silently swallow errors — if a catch block is empty, add a comment explaining why.
- Narrow error types at boundaries — don't let `unknown` leak into your domain.
- Explicitly type Promise return values: `Promise<UserDTO>` not `Promise<any>`.

## Design Patterns

Apply patterns to solve real problems — never to demonstrate pattern knowledge:

- **Factory**: when object creation is complex or varies by configuration.
- **Strategy**: when you need interchangeable algorithms with the same interface.
- **Repository**: when you want to decouple data access from business logic.
- **Adapter**: when integrating third-party libraries that don't match your domain interface.
- **Observer / EventEmitter**: for decoupled event-driven communication.
- **Builder**: for constructing complex objects with many optional parameters.

If a simpler approach works, use it. Three lines of inline code beat an abstract factory with one use case.

## Async / Concurrency

- Use `async/await` consistently — avoid mixing with raw `.then()/.catch()` chains.
- Run independent async operations in parallel with `Promise.all` or `Promise.allSettled`, not sequentially with `await` in a loop.
- Handle both the happy path and rejection in every async operation.

## Testing

- Name tests as sentences describing behaviour: `it('returns null when the input is empty')`.
- Follow **Arrange-Act-Assert** (AAA) with a blank line between each phase.
- Test the contract (observable behaviour), not the implementation (internal structure).
- Use typed mocks — mock types must match the real interface exactly.
- One behaviour per test — keep tests focused and independent.
- Cover: happy path, edge cases (empty, null, boundary values), and error/failure scenarios.
- Use `beforeEach`/`afterEach` for setup/teardown; avoid shared mutable state between tests.
- Prefer `vi.fn()` (Vitest) with proper generic typing over untyped mocks.

## Self-Review Checklist

Before finalising code, verify:
- [ ] No `any` — all types are explicit and accurate
- [ ] Edge cases handled (null, undefined, empty collections, network failures)
- [ ] No code duplication
- [ ] Error handling is explicit and typed
- [ ] Imports are clean — only what is used
- [ ] No unused variables, parameters, or imports
- [ ] Tests: every assertion is meaningful and would catch a real regression
- [ ] Pattern usage is justified — no over-engineering
- [ ] No `else`/`else if` after a `return`, `throw`, or `continue` — control flow is flattened with early exits
- [ ] No magic numbers or magic strings — all literals extracted into named constants

---
name: coding-standards
description: Use whenever an agent writes, edits, fixes, implements, refactors, reviews, formats, tests, or changes dependencies for code. Provides baseline coding standards that all code work must follow, with optional routing to more specific rule files when available.
origin: user
---

# Coding Standards

This skill defines the baseline coding standards agents must follow when writing, modifying, reviewing, or refactoring code.

It is the shared floor for code quality. Project-specific conventions, framework-specific skills, README instructions, formatter configuration, and CI rules take precedence when they are more specific.

## When to Activate

Use this skill when:

- Starting a new project, module, file, or feature.
- Modifying existing code and needing to preserve project style.
- Reviewing code for readability, maintainability, naming, formatting, or structure.
- Refactoring code to reduce complexity or remove code smells.
- Adding or changing dependencies, formatters, linters, type checkers, or test commands.
- Writing Python, JavaScript, TypeScript, TSX, HTML, C++, or Rust.

## Scope Boundaries

This skill covers:

- Naming, formatting, imports, comments, and file organization.
- Simplicity, readability, type clarity, explicit boundaries, and testability.
- Python package-management defaults with `uv`.
- General error handling, dependency discipline, and code-smell detection.

This skill does not replace narrower project or framework guidance. If a dedicated frontend, backend, API, database, security, or framework skill exists, use that skill together with this baseline.

## Priority Order

When standards conflict, follow this order:

1. Existing project configuration, CI, README, and established local style.
2. Framework-specific or domain-specific skill instructions.
3. Language-specific sections in this skill.
4. General principles in this skill.

Do not introduce a new architecture, directory structure, formatter, package manager, or dependency style unless the project explicitly needs it.

## Universal Code Quality Principles

### Readability First

- Code is read more often than it is written.
- Prefer clear names, explicit data flow, stable formatting, and direct control flow.
- Self-documenting code is better than comments that restate the obvious.
- Comments should explain why something exists, not what the code literally does.

### KISS (Keep It Simple, Stupid)

- Choose the simplest solution that correctly solves the current problem.
- Avoid cleverness, excessive indirection, and hidden behavior.
- Prefer ordinary functions and explicit branches over unnecessary design patterns.
- Do not sacrifice readability for small or unproven performance gains.
- Easy to understand > clever code
- Avoid over-engineering

### DRY (Don't Repeat Yourself)

- Extract common logic when duplication is real, stable, and meaningful.
- Local clear repetition is better than premature abstraction.
- Do not create vague abstractions named `Manager`, `Handler`, `Processor`, or `Util` unless their responsibility is precise.
- Share utilities across modules
- Avoid copy-paste programming

### YAGNI (You Aren't Gonna Need It)

- Do not build features, extension points, configuration layers, or abstractions for hypothetical future needs.
- Add complexity only when the current requirement proves it is necessary.
- Start simple and refactor when real repetition or pressure appears.

### Correctness Before Performance

Use this priority order:

1. Correctness.
2. Readability.
3. Testability.
4. Extensibility.
5. Performance.

Optimize only after a bottleneck is demonstrated by tests, metrics, profiling, or production evidence.

## Working in Existing Code

Before changing existing code, understand:

- Where the entry points are.
- Where data enters, is validated, and is persisted.
- Where business logic belongs.
- How tests cover current behavior.
- Which frameworks, tools, naming conventions, and file structure already exist.

Do not rewrite broad structure when a local change solves the task. Prefer extending existing interfaces, adding clear cases, and keeping diffs focused.

## General Structure and Logic

- Keep functions, classes, and modules focused on one responsibility.
- Avoid mixing input parsing, validation, business logic, persistence, network calls, and presentation in one function.
- Use early returns for invalid input and special cases.
- Avoid deep nesting; split complex conditions into named booleans or helper functions.
- Use named constants for magic numbers and magic strings.
- Keep side effects explicit and concentrated at boundaries.
- Do not perform important side effects at module import time.
- Validate and convert external input at system boundaries before passing it into core logic.

## Naming

- Names should express business meaning, not just type or shape.
- Avoid broad names such as `data`, `obj`, `item`, `temp`, `flag`, `x`, and `result` unless the scope is tiny and obvious.
- Boolean names should read like predicates: `is_active`, `hasPermission`, `shouldRetry`.
- Functions should usually use verb-noun or intent-revealing names such as `fetchMarketData`, `calculate_total`, or `validate_email`.
- Side-effecting functions should make the side effect visible, such as `save_user`, `sendEmail`, or `write_report`.

## Comments and Documentation

- Explain why, not what.
- Remove stale comments, commented-out code, debug output, and temporary notes.
- Preserve AI provenance and AI-edit boundary comments. Do not delete comments similar to `// Assisted-by: AGENT_NAME:MODEL_VERSION [TOOL1] [TOOL2]`, `// <<<AI START>>>`, or `// <<<AI END>>>` unless the user explicitly asks to remove those markers.
- Public APIs, public classes, important business rules, non-obvious constraints, side effects, and error behavior may need concise documentation.
- Internal helpers with clear names and types do not need boilerplate comments.

Document functions when any of these conditions apply:

- The function body is longer than 10 lines.
- The function has more than 3 parameters.
- The function dispatches work, schedules work, or routes tasks to handlers.
- The function has asynchronous behavior.
- The function acquires, releases, waits on, or otherwise coordinates locks.
- The function performs indirect calls, dynamic dispatch, or virtual-table-backed calls.
- The function creates, coordinates, awaits, joins, or otherwise participates in threads, coroutines, or comparable concurrent execution.

For these functions, explain the function's purpose, key control-flow decisions, side effects, concurrency or dispatch invariants, and non-obvious failure behavior. Keep the explanation concise and remove it when refactoring makes the function simple again.

### Structural Separator Comments

- Add a separator comment before every class, every top-level function, and every group of consecutive methods inside a class.
- The separator uses `----` plus the class, function, or method-group name, extended to a full line with `-`.
- Use the language's normal comment syntax, for example `// ---- SessionManager ---------------------------------------------------------` or `# ---- load_user --------------------------------------------------------------`.

## Error Handling

- Preserve useful context when raising or rethrowing errors.
- Do not swallow errors silently unless that is explicit business behavior.
- Do not catch every error and convert it to a vague generic error.
- Distinguish common external-system failures such as timeout, authentication failure, rate limiting, missing data, and invalid response shape.
- MUST: When code emits or handles errors, make the error cause as explicit as possible and identify where the error occurred so an LLM can quickly locate the code to fix.
- Error messages should help locate the problem without leaking passwords, tokens, cookies, secrets, or personal sensitive data.

## Dependency Discipline

- Prefer the standard library and existing project dependencies before adding new dependencies.
- Add a dependency only when it clearly reduces complexity or provides necessary capability.
- Avoid dependencies for small logic that is easy to implement locally.
- Check that new dependencies are maintained and do not significantly complicate install, build, or runtime.

## Python Standards

When writing or modifying Python, read and follow these repository standards before editing:

- `language-rules/python/format.md` for formatting, imports, naming, type annotations, functions, classes, comments, logging, and pre-submit checks.
- `language-rules/python/logic.md` for code logic, simplicity, boundaries, data modeling, dependency discipline, testability, side effects, configuration, errors, abstraction, async, and performance.
- `language-rules/python/uv.md` for package management, dependency changes, command execution, lockfile handling, and package-manager exceptions.

Use all three documents for Python feature work. For a narrow formatting-only change, `language-rules/python/format.md` is sufficient unless dependencies, tests, or execution commands are involved.

## JavaScript Standards

When writing or modifying JavaScript, read and follow `language-rules/js/format.md` before editing. It covers formatting, Prettier defaults, imports and exports, naming, control-flow layout, object and array formatting, DOM/event code, comments, and LLM-readable structure.

## TypeScript Standards

When writing or modifying TypeScript or TSX, read and follow `language-rules/ts/format.md` before editing. It covers formatting, Prettier defaults, imports and exports, naming, type syntax, functions, control flow, object and type-literal formatting, TSX layout, comments, and LLM-readable structure.

If the file is plain JavaScript, use `language-rules/js/format.md` instead. If a TypeScript task also touches HTML templates, read `language-rules/html/format.md` as well.

## HTML Standards

When writing or modifying HTML, read and follow `language-rules/html/format.md` before editing. It covers document structure, tag layout, attribute order, class and id naming, forms, templates, scripts, comments, and Prettier defaults.

If HTML work includes JavaScript modules or inline script changes, also read `language-rules/js/format.md` or `language-rules/ts/format.md` according to the language used.

## C++ Standards

When writing or modifying C++, read and follow `language-rules/cpp/format.md` before editing. It covers formatting, include order, naming, pointer and reference style, functions, classes and structs, comments, LLVM-specific conventions, and validation checks.

## Rust Standards

When writing or modifying Rust, read and follow these repository standards before editing:

- `language-rules/rust/format.md` for rustfmt defaults, file and module layout, imports, naming, expression layout, comments, separator comments, tests, and formatting checks.
- `language-rules/rust/logic.md` for project constraints, ownership and parameter design, public API shape, error handling, `unsafe` boundaries, visibility, and validation expectations.

Use both documents for Rust feature work. For a narrow formatting-only change, `language-rules/rust/format.md` is sufficient unless API shape, ownership, errors, tests, `unsafe`, or validation behavior is involved.

## Immutability and State

- Prefer immutable updates when changing objects and arrays, especially in JavaScript, TypeScript, React, and stateful code.
- Use spreads, mapping, filtering, and new objects instead of mutating shared state.
- Mutation is acceptable only when it is local, intentional, and clearer or measurably necessary.

Good:

```ts
const updatedUser = {
  ...user,
  name: "New Name",
};

const updatedItems = [...items, newItem];
```

Avoid:

```ts
user.name = "New Name";
items.push(newItem);
```

## Async and Concurrency

- Use async or concurrency only when there is a clear need for I/O concurrency, background work, throughput, or responsiveness.
- Do not convert synchronous code to async without reason.
- Keep async call chains consistent; avoid mixing blocking I/O into async functions.
- Concurrent tasks should have timeout, cancellation, and error-handling strategy.
- Run independent async operations in parallel when it is safe.

Example:

```ts
const [users, markets, stats] = await Promise.all([
  fetchUsers(),
  fetchMarkets(),
  fetchStats(),
]);
```

## Testing Standards

- Add or update tests when behavior changes and the project has a test structure.
- Do not add a new test framework to a project that has no tests unless explicitly asked.
- Tests should cover normal behavior, boundary cases, and meaningful error paths.
- Test names should describe behavior, not implementation details.
- Prefer Arrange, Act, Assert structure when it improves clarity.

Example:

```ts
test("returns empty array when no markets match query", () => {
  const markets = [{ id: "1", name: "Election" }];

  const result = searchMarkets(markets, "weather");

  expect(result).toEqual([]);
});
```

## Validation Before Finishing

Before declaring work complete:

- Run the most specific formatter, linter, type checker, and tests available for the changed code.
- Then widen validation if the change affects shared code or public behavior.
- Prefer project-defined commands from README, package scripts, pyproject, CI, or task runner.
- If validation cannot run, state exactly why.

Typical commands:

```bash
uv run ruff format .
uv run ruff check .
uv run pytest
npm test
npm run lint
npm run typecheck
```

## Code Smells to Reject

Watch for and fix code smells introduced by the change:

- Long functions that mix multiple responsibilities.
- Deep nesting that can be replaced with early returns.
- Magic numbers or magic strings without names.
- Vague names that hide business meaning.
- Unused imports, unused variables, dead code, and commented-out code.
- Unnecessary classes, inheritance, registries, or abstractions.
- Hidden side effects or important work happening at import time.
- Broad exception handling that loses context.
- Repeated environment-variable reads scattered through business logic.
- New dependencies that solve only a tiny local problem.

## Final Checklist for Agents

Before handing back code, confirm:

- The implementation follows existing project style and toolchain.
- The simplest correct design was used.
- Names are descriptive and consistent.
- Types or data models make boundaries clear.
- External input is validated at the boundary.
- Side effects are explicit and testable.
- Errors preserve context and avoid leaking secrets.
- Dependencies, if changed, were managed with the project's package manager.
- Relevant formatter, lint, type-check, and test commands were run or clearly reported as unavailable.

# Code Quality & Structure

## TypeScript

- `strict: true` in `tsconfig.json`, no exceptions by default.
- `any` is a last resort with a comment explaining why, never a quiet shortcut past a type
  error.
- Model real domain concepts as types (a branded `UserId` instead of a bare `string`) when
  mixing up two similarly-shaped values would cause an actual bug.

## Structure

- One clear responsibility per file/module. If a file mixes data-fetching, business logic,
  and rendering with no separation, split it.
- Colocate code that changes together (a feature's components, hooks, and API calls) over
  grouping strictly "by type" once a project passes toy size.
- Shared utilities live in an obviously-named shared location — logic shouldn't be
  duplicated across three files because copy-paste was faster in the moment.

## Error handling

- Every `await` that can reject is in a `try/catch` or has an explicit `.catch` — no
  fire-and-forget promises.
- Distinguish expected failures (validation error, not-found) from unexpected ones (a
  crashed dependency): the former returns a clean, typed error/response; the latter gets
  logged and surfaces a generic message to the user, never a stack trace.
- Fail loudly in development, gracefully in production.

## Naming & readability

- Names describe intent, not type (`activeUsers`, not `arr2`).
- A function does one thing. If its name needs "and" to describe it, split it.
- Comments explain *why*, not *what*. If a comment just restates the code below it, delete
  it. If a decision looks wrong out of context — a workaround, a magic number, an
  unintuitive ordering — that's exactly when a comment earns its place.

## Linting & formatting

ESLint + Prettier (or Biome as a faster combined alternative), configured and passing from
the first commit — not bolted on later. When scaffolding a project, include the config
files, not just the code.

## Avoiding both failure modes

- **Under-engineering:** no error handling, no types, everything in one file, magic strings
  everywhere.
- **Over-engineering:** five layers of abstraction, a factory for a factory, a generic
  plugin system for a feature with exactly one implementation. A todo app doesn't need a
  repository-pattern-with-dependency-injection-container. Match structure to the
  complexity that actually exists, not the complexity someone imagines might show up later.

## Avoiding the tells that make code look AI-generated

Bugs aside, there's a stylistic layer that's its own kind of slop — the thing that makes a
senior engineer wince before they've even checked whether the logic is correct:

- No filler comments that just restate the line below them (`// increment i by 1`,
  `// return the result`). If a comment doesn't add information the reader didn't already
  have, delete it.
- No leftover scaffolding — no unused imports, no commented-out old code, no placeholder
  names like `doStuff` or `handleThing2` that survived from a first draft.
- Consistent naming and formatting across a whole file or feature, not camelCase in one
  function and snake_case in the next because they were generated in different passes.
- Consistent error shapes — every error response follows the same structure
  (`{ error: { code, message } }` or whatever the project already uses), not a different ad
  hoc shape per endpoint.
- Prefer a clear multi-line implementation over a "clever" nested ternary or heavily
  chained one-liner that saves three lines and costs the reader thirty seconds.
- Pick one async pattern (`async`/`await`) and use it consistently — don't mix in raw
  `.then()` chains or callback-style code in the same file without a real reason.
- If a pattern already exists in the codebase for this exact purpose — a hook, a helper, a
  validation function — use it. Don't quietly introduce a second, slightly different way
  of doing the same thing.

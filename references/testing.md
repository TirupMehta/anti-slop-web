# Testing

## Minimum bar by project type

- **Prototype/demo:** no tests required — but say so explicitly rather than silently
  skipping them.
- **Anything with real users:** unit tests on business logic, plus at least one
  integration/e2e test per critical user flow (signup, login, checkout — whatever the
  app's core action is).
- **Anything touching money, auth, or user data:** the above, plus tests for the
  failure/edge paths — wrong password, expired token, duplicate submission, insufficient
  permissions — not just the happy path.

## Tools

- Unit/integration: Vitest.
- Component tests: React Testing Library — test behavior, not implementation. Don't assert
  on internal state or CSS class names.
- End-to-end: Playwright.

## What to actually test

- Business logic and validation rules — cheap to test, catches real bugs.
- API contracts — request in, response out, including error responses.
- Critical user flows end-to-end — the few paths where a bug means a user can't do the
  core thing the app exists for.

## What not to bother testing

- Trivial getters/setters, third-party library internals, static markup with no logic.
- Don't chase a coverage percentage as a goal in itself. A suite of shallow tests written
  just to hit 90% coverage is its own kind of slop.

## CI gate

Lint, typecheck, test, and build should all run on every pull request before merge — not
just on a developer's machine "when they remember."

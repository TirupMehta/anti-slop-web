# Observability & Operations

## Logging

- Structured logs (JSON), not `console.log` scattered through the codebase, for anything
  beyond a local script.
- Log enough to debug a production incident (request id, user id if relevant, error stack)
  without logging secrets, passwords, tokens, or full payloads containing personal data.

## Error handling & monitoring

- Unhandled exceptions and rejected promises are caught centrally (a global error
  handler/middleware) so nothing fails silently.
- Wire up error tracking (or at minimum, structured logs an operator can actually search)
  before real users show up. "We'll add monitoring later" is how outages go unnoticed.

## Environments

- Distinct dev/staging/prod configuration, with production secrets never available in
  dev or staging.
- Feature flags or environment checks for anything risky being rolled out gradually,
  rather than an `if (isDev)` scattered through business logic.

## Deployment hygiene

- CI runs lint, typecheck, tests, and build on every PR (see `testing.md`); nothing merges
  to main on a red pipeline.
- Database migrations are versioned and applied through a migration tool — never a
  hand-run `ALTER TABLE` against production.
- A rollback plan exists (a previous build/image stays deployable) before shipping
  anything risky.

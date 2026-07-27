# Architecture & Future-Proofing

## Project structure

- A new project gets the conventional structure for its framework (e.g. Next.js App Router
  conventions) rather than an invented one — familiarity is part of maintainability.
- Separate concerns: data access, business logic, and presentation shouldn't be tangled in
  one file, so any one of them can change or be replaced without touching the others.

## Configuration

- Environment-specific config (dev/staging/prod) lives in environment variables or
  per-environment config files — never hardcoded conditionals sprinkled through business
  logic (`if (url.includes('localhost'))`).
- One source of truth for schema — don't let the database schema, the ORM models, and the
  validation schema drift out of sync. Generate one from another where the tooling
  supports it.

## Dependencies & upgrades

- Pin dependency versions sensibly and commit the lockfile. Don't pin so tight that
  security patches never land; don't leave everything on `latest` so an unrelated minor
  bump breaks the build unexpectedly.
- Prefer boring, widely-adopted, actively-maintained libraries over whatever's trending
  this week. "Future-proof" usually means "still maintained in three years," not "has the
  most GitHub stars right now."
- When a project depends on something with a documented end-of-life or maintenance-mode
  status (see `stack-defaults.md` for current examples), flag it and suggest a migration
  path rather than silently building more on top of it.

## Avoiding lock-in without being paranoid about it

Depending on a managed service (a database host, an auth provider) is fine — the real risk
is only if switching later would require a full rewrite. Keep the integration behind a
reasonably thin interface where that's cheap to do. Don't build an elaborate abstraction
layer "just in case" for a dependency that would take an afternoon to swap anyway — that's
the over-engineering failure mode from `code-quality.md` wearing a future-proofing costume.

## API versioning

Any API with external consumers gets a versioning strategy from day one (URL- or
header-based). Breaking a consumer with an unversioned change is a future-proofing
failure, not a minor bug.

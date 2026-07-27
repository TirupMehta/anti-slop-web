# Performance

## Database

- Index columns used in `WHERE`, `JOIN`, and `ORDER BY` clauses — a lookup that's instant
  at 100 rows falls over at 100k.
- Watch for N+1 queries: fetching a list, then querying per item in a loop. Use a join or a
  batched/`IN` query instead.
- Paginate anything that can grow unbounded. Never `SELECT *` on a table with no row limit.
- Use connection pooling — don't open a new DB connection per request in a serverless
  environment without a pooler.

## Frontend

- Code-split by route by default (modern frameworks like Next.js do this automatically —
  don't undo it by importing everything into one entry bundle).
- Serve images in modern formats, sized appropriately, lazy-loaded below the fold (e.g.
  `next/image` or equivalent) — not a raw `<img>` at full source resolution.
- Avoid re-render storms: memoize expensive computations and components where profiling
  actually shows it's needed. See `code-quality.md` on over-engineering — don't wrap every
  component in `memo` speculatively.

## Caching

- Cache expensive, repeatable reads (HTTP cache headers, a CDN, or an in-memory/Redis
  cache) instead of recomputing on every request.
- Set explicit cache invalidation when the underlying data changes. A cache with no
  invalidation story is a bug waiting to be filed as "stale data."

## The actual rule

Measure before optimizing. Don't hand-tune a query or micro-optimize a component that's
never been profiled — but do avoid the well-known footguns above by default, since they're
cheap to avoid up front and expensive to retrofit once real traffic shows up.

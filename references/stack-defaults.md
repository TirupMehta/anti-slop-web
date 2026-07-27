# Stack Defaults (JS/TS/React/Next.js/Node)

> **Snapshot: mid-2026.** This ecosystem moves fast — treat this as a strong starting
> point, not gospel. Before locking a new project into a major dependency (auth, ORM,
> framework), do a quick check that it's still actively maintained: recent releases or
> commits, no deprecation notice, not sitting in "maintenance mode, security patches only."
> Repeating a stale recommendation past its expiry date is one of the most common ways AI
> output turns into slop — the code runs today and quietly becomes a liability in a year.

## Language

TypeScript, strict mode, always. Plain JS is only acceptable for tiny one-off scripts with
no shared state.

## Frontend / full-stack framework

- **Default: Next.js (App Router) + React.** Still the dominant choice for production React
  apps — largest ecosystem, best hosting support, easiest to hire for.
- Component-only SPA with no server needs: Vite + React.
- Content-heavy site with light interactivity (blog, docs, marketing): consider Astro
  before defaulting to a full React app — it ships far less JS by default.

## Backend / API

- **Default: Express** for straightforward REST APIs. Enormous ecosystem, battle-tested,
  easiest to find help and hires for.
- Edge-first or performance-critical: Hono or Fastify.
- Larger team wanting enforced structure, DI, and conventions: NestJS.
- Don't reach for whatever's trendiest this month over something boring and
  battle-tested without a concrete reason (edge deployment, extreme scale, team preference).

## Database access / ORM

- **Default: Drizzle** for new projects — SQL-first, small bundle, works natively at the
  edge/serverless, no code-generation step.
- Prisma is still a legitimate choice, especially when the team is less comfortable with
  raw SQL, wants the visual data browser (Prisma Studio), or is optimizing for speed of
  initial build over long-term bundle size and cold-start latency.
- Never hand-build SQL strings for anything touching user input. Use the ORM's
  parameterized query builder, or Kysely if you want SQL-level control with type safety.

## Auth

- **Default: Better Auth** for new self-hosted projects — actively developed, the app owns
  its own auth data, ships 2FA/passkeys/RBAC without bolting on extra libraries.
- Want managed/hosted convenience over data ownership: Clerk (or WorkOS for B2B apps that
  need enterprise SSO).
- Auth.js / NextAuth: fine to maintain on an existing project, but as of this snapshot it's
  in maintenance mode (security patches only, no new feature development) — don't start a
  *new* project on it without checking whether that's changed.
- Never hand-roll session or password logic from scratch. This is exactly where
  AI-generated code tends to quietly reinvent a worse, less secure version of a solved
  problem.

## Validation

Zod for schema validation and type inference at every boundary — API input, form input,
environment variables.

## Testing

Vitest for unit/integration tests, Playwright for end-to-end, React Testing Library for
component tests. See `testing.md` for what's actually worth testing.

## Styling

Tailwind CSS as a low-friction default, unless the project already has an established
design system to work within.

## The actual rule, if nothing else

When a user hasn't specified a stack, don't hand them a wall of options and a paragraph of
"it depends." Recommend the default above with one sentence of *why*, and move on.
Indecision dressed up as thoroughness is its own kind of slop. If they push back or their
project has a specific constraint (edge-only hosting, an existing non-SQL database,
whatever), adapt from there.

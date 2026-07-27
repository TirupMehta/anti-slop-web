---
name: anti-slop-web
description: Mandatory skill whenever writing, generating, reviewing, refactoring, or scaffolding web application code — components, pages, forms, APIs, Next.js Server Actions, database access, auth, WebSockets, AI integrations, or full apps — in JavaScript/TypeScript, React, Next.js, or Node. Enforces production-grade security (OWASP), server-side input validation, authorization boundaries, data-layer idempotency, multi-tenant isolation, serverless connection pooling, boot-time env schemas, accessibility, and agent self-verification.
---

# Production-Grade Web Development (anti-slop-web)

## Why this exists

AI-generated web code usually runs. That is a low bar. It consistently omits the unspoken judgments senior engineers make automatically: real input schema validation, resource-level authorization (not just authentication), sane error masking, boot-time env schemas, idempotency for state mutations, and serverless connection management.

This skill forces the exact same production checklist a skeptical staff engineer runs during code review. Write every line as if it is about to be audited line-by-line by a security lead.

---

## The Non-Negotiables

These apply to *every* piece of code this skill touches, regardless of project size:

1. **Zero hardcoded secrets**: All credentials, keys, and tokens live in environment variables or secret managers.
2. **Parameterized database access**: Zero string-concatenated SQL queries or raw dynamic commands.
3. **Strong password hashing**: `argon2id` or `bcrypt` (cost ≥ 12).
4. **Server-side validation**: External inputs must be parsed against a Zod schema server-side. Client validation is UX only.
5. **Masked stack traces**: Internal paths and raw database errors must never reach client responses.
6. **Explicit authorization boundaries**: Every mutating route/endpoint/action MUST verify resource ownership (`userId` / `workspaceId`), not just login status.
7. **Server Action security**: Next.js Server Actions (`"use server"`) are public HTTP POST endpoints — parse inputs and check authorization inside every single action body.
8. **RSC payload protection**: Never pass raw database objects to Client Components (`"use client"`). Project explicit DTOs.
9. **Boot-time environment validation**: Fail fast on app startup if required environment variables are missing using boot schemas (`@t3-oss/env-nextjs` / `@t3-oss/env-core`).
10. **Multi-tenant query scoping**: Dual-scope every query with `workspaceId` / `tenantId` at the data layer, including nested relations.
11. **Serverless connection pooling**: Use connection poolers or global singletons for serverless DB drivers to prevent connection exhaustion.
12. **AI API key & cost protection**: Never expose AI API keys on the client (`NEXT_PUBLIC_*`). Enforce per-user rate limits and `max_tokens` generation caps on all LLM calls.
13. **Maintained dependencies**: Flag deprecated, unmaintained, or vulnerable dependencies before adding them.
14. **Strict TypeScript**: TypeScript strict mode is on. No unannotated `any` types.
15. **Handled async promises**: Zero unhandled or swallowed promises in catch blocks.
16. **Complete UI states**: Data-fetching UIs must handle loading, empty, error, and optimistic rollback states.
17. **Data integrity & idempotency**: State-mutating operations (payments, orders, cancellations) protected at the data layer against duplicate execution.
18. **Honest confidence & self-verification**: Run type checks (`tsc --noEmit`) and linter before declaring tasks complete. Report unverified gaps explicitly.

---

## Agent Self-Verification Protocol

Before declaring any coding task complete or presenting code as "production-ready":

1. **Self-Audit**: Verify every line against the 18 Non-Negotiables above.
2. **Execute Type Check**: If shell commands are available in your agent environment, run `npx tsc --noEmit` or the workspace typecheck script to verify zero TypeScript errors.
3. **Execute Lint / Build**: Run `npm run lint` or `npm run build` if modifying core configuration or routing.
4. **Report Unverified Items**: State explicitly any item that could not be verified in the local environment (e.g., pending database migrations, external OAuth secret setup).

---

## Task Routing & Reference Manuals

Route to the relevant reference document based on the task:

| Task Type | Reference Manual | Primary Focus |
|---|---|---|
| New Component / UI Page | `references/code-quality.md`, `references/accessibility-design.md` | WCAG AA compliance, state management, micro-interactions |
| Next.js Server Actions & RSC | `references/nextjs-server-actions.md` | Public POST security, RSC payload DTOs, server-only isolation |
| API Endpoint / Backend Logic | `references/security.md`, `references/code-quality.md`, `references/testing.md` | OWASP Top 10, authorization boundaries, edge testing |
| AI / LLM Integrations | `references/ai-llm-integrations.md` | Key protection, rate limiting, token caps, streaming errors |
| WebSockets / Real-Time | `references/realtime-websockets.md` | Handshake auth, room boundaries, unmount cleanup |
| Environment Setup | `references/environment-boot-validation.md` | Boot-time schema validation, client/server secret separation |
| Multi-Tenant & Serverless DB | `references/multi-tenant-data-isolation.md` | Workspace scoping, soft-deletes, connection pooling |
| State Mutations & Money | `references/data-integrity.md` | Idempotency keys, atomic updates, transaction safety |
| Code Audit / Review Mode | `references/review-mode.md`, `references/loopholes.md` | Catching subtle slop, rationalizations, and TODO stubs |
| Project Scaffolding | `references/stack-defaults.md`, `references/architecture-future-proofing.md` | Modern tech stack, clean abstractions, deployment ops |

---

## Worked Examples

* `examples/secure-api-endpoint.md` — Secure backend endpoint with authorization and Zod validation.
* `examples/accessible-form-component.md` — Accessible React form with loading/error state management.
* `examples/review-mode-before-after.md` — Concrete review mode audit output.

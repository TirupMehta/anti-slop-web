---
name: anti-slop-web
description: Mandatory skill whenever writing, generating, reviewing, refactoring, or scaffolding web application code — components, pages, forms, APIs, database access, auth, or full apps — in JavaScript/TypeScript, React, Next.js, or Node. This applies even to small or "quick" requests like a landing page, a single CRUD endpoint, or a login form — those are exactly where quality gets skipped. Also use this skill whenever the user asks to review, audit, harden, or clean up existing code (especially AI-generated code), or asks what library/framework/tech to use for a web project. Always consult it before calling web code done — it enforces production-grade security, code quality, accessibility, performance, testing, and maintainability instead of prototype-quality defaults.
---

# Production-Grade Web Development

## Why this exists

AI-generated web code usually runs. That's a low bar. It's usually missing the things a
senior engineer applies automatically without being asked: real input validation, real
authorization checks (not just "are you logged in"), sane error handling, basic
accessibility, and dependency choices that are still maintained next year. This skill's
job is to make that judgment mandatory — not to teach new syntax, but to force the same
checklist a good senior engineer runs in their head, every time, even on "quick" requests.

Write every line as if a skeptical senior engineer is about to review it line by line and
has no reason to be generous. Not because they're hostile — because that's the actual bar
production code needs to clear, and it's a more useful target than "would this pass a
casual glance."

## The Non-Negotiables

These apply to *every* piece of code this skill touches, regardless of project size,
regardless of whether the user asked for them. A prototype can skip a test suite. Nothing
skips this list.

1. No hardcoded secrets, API keys, or credentials — ever, even in examples. Use env vars.
2. No string-concatenated queries. Parameterize everything that touches a database.
3. Passwords are hashed with argon2id or bcrypt (cost ≥ 12). Never plaintext, never MD5/SHA1,
   never reversible encryption for a password.
4. Every external input is validated **server-side** against a schema. Client-side
   validation is UX, never the security boundary.
5. Errors shown to users never leak stack traces, internal paths, or raw DB errors.
6. Every mutating endpoint checks not just "is someone logged in" but "is *this* user
   allowed to act on *this* resource." (The classic slop bug: any logged-in user can edit
   any other user's data by changing an ID in the request.)
7. Every dependency added is current and actually maintained — check before adding one,
   not after. Flag anything abandoned, deprecated, or in maintenance-only mode.
8. TypeScript strict mode is on. `any` requires a comment explaining why, not silence.
9. Every `await` that can fail is handled — no silently swallowed promises.
10. Any UI that fetches data has a loading state, an empty state, and an error state — not
    just the happy path.
11. Security headers and CORS are explicitly set and reviewed — never left at whatever the
    framework defaults to without checking what that actually is.
12. Nothing ships without at least one test covering its critical path.
13. Confidence in your response matches what was actually verified. Never tell the user
    something is "production-ready," "fully secure," or "tested" unless every part of that
    claim is true. State what wasn't done as plainly as what was — an honest gap list is
    worth more than an unearned checkmark.
14. Anything that must not happen twice — a payment, an order, an email send, provisioning
    a resource — is protected against duplicate execution at the data layer (a unique
    constraint, an idempotency key, an atomic conditional update), not just a disabled
    submit button. See `references/data-integrity.md`.

## Where this quietly falls apart

A checklist only works if it can't be rationalized around. In practice, code quality
degrades back toward slop through a handful of specific loopholes, even while technically
"following" everything above:

- **Unearned scope-shrinking** — deciding something is "just a prototype" to justify
  skipping the Non-Negotiables, when the user never actually said that.
- **Checklist theater** — a catch block that only logs and swallows the error, a validator
  that checks `typeof x === 'string'` and calls it done, a rate limiter set so loose it
  never actually triggers on the endpoint that needed it most. The checkbox gets ticked
  without the substance behind it.
- **TODO-stubbing** — `// add validation later` presented as if the task were finished.
- **Unearned confidence** — telling the user something is secure/tested/production-ready
  without having actually verified it (see Non-Negotiable 13).
- **Incremental drift across turns** — a feature built up over several small messages
  never gets audited as a whole, even though each individual message looked too small to
  bother with a full check.
- **Grading your own homework leniently** — reviewing code from earlier in the same
  conversation with a softer eye than a stranger's code would get.

Full catalog with concrete tells and fixes: `references/loopholes.md`. Worth rereading
deep into a long session, or before reviewing your own earlier output — that's exactly
when these creep back in.

## How to use this skill

**Step 1 — route to the right reference file(s) based on the task:**

| Task | Read |
|---|---|
| New component / page / UI | `references/code-quality.md`, `references/accessibility-design.md` |
| New API endpoint or backend logic | `references/security.md`, `references/code-quality.md`, `references/testing.md` |
| Full app / project scaffold | `references/stack-defaults.md`, `references/architecture-future-proofing.md`, `references/security.md`, `references/observability-ops.md` |
| Database work | `references/security.md` (injection/access), `references/performance.md` (indexing/N+1) |
| Anything that changes state and can't safely run twice (orders, payments, sending email, provisioning) | `references/data-integrity.md`, in addition to whatever else applies |
| "What should I use for X?" / picking a library | `references/stack-defaults.md` |
| Reviewing, auditing, or hardening existing code | `references/review-mode.md` first, then whatever else applies |
| Anything that doesn't clearly match a row above (a scraper, a webhook receiver, a cron job) | Default to `references/security.md` + `references/code-quality.md` — don't treat "not in the table" as "nothing applies." |

**Step 2 — scale the bar to the project's actual stakes, never below the Non-Negotiables.**
Default assumption is that code might end up handling real users and real data. Only relax
rigor below the Non-Negotiables floor when the *user* has explicitly said this is a
prototype/demo/throwaway — never assume that on the code's behalf to save effort. Even
then, state out loud what's being relaxed and why ("skipping tests here since you said this
is a throwaway demo") rather than silently cutting corners, and the Non-Negotiables
themselves (secrets, injection, password hashing, authorization, leaked errors) still never
move, prototype or not.

**Step 3 — before handing code back, go through the Non-Negotiables one line at a time
against the actual code, not from memory of having followed them while writing.** This is a
distinct pass, not a mental "I was careful" impression — that's exactly the gap the
loopholes above live in. If something was skipped, say so explicitly and why, instead of
presenting incomplete work as finished.

## Review mode

If the user asks you to review, audit, clean up, or "make production-ready" existing code
— including code generated earlier in the same conversation — don't just silently rewrite
it. Go to `references/review-mode.md` for the process and output format. The point is that
the person understands what was wrong, not just that the code changed.

## Worked examples

`examples/` has a few small, fully-annotated, gold-standard samples in this stack: a
complete API endpoint (`examples/secure-api-endpoint.md`), an accessible form component
(`examples/accessible-form-component.md`), and a before/after showing review mode's
expected tone and output shape (`examples/review-mode-before-after.md`). Reading one before
generating something structurally similar for the first time in a session is a faster way
to calibrate than re-reading every reference file's prose.

## Reference files

- `references/stack-defaults.md` — opinionated JS/TS/React/Next.js/Node tech choices, with
  reasoning, for when the user hasn't specified a stack.
- `references/security.md` — the OWASP-style baseline: auth, input handling, secrets,
  headers, rate limiting, dependency hygiene.
- `references/code-quality.md` — TypeScript config, structure, naming, error handling,
  avoiding both under- and over-engineering.
- `references/testing.md` — what to test, what not to bother testing, tools, CI gating.
- `references/performance.md` — DB indexing/N+1, frontend bundle/caching, when to actually
  optimize.
- `references/data-integrity.md` — race conditions, idempotency, transactions, and
  concurrent-edit handling for anything that mutates state.
- `references/accessibility-design.md` — WCAG-AA baseline plus the design details
  AI-generated UIs usually skip (empty states, confirmation on destructive actions, etc).
- `references/architecture-future-proofing.md` — project structure, config management,
  dependency pinning, avoiding lock-in without over-engineering against it.
- `references/observability-ops.md` — logging, error tracking, environments, deployment
  hygiene.
- `references/review-mode.md` — how to audit and prioritize findings in existing code.
- `references/loopholes.md` — the specific ways this checklist gets satisfied on paper but
  not in substance, with concrete tells and fixes for each.

## Scope note

This skill is currently tuned for the JS/TS ecosystem (React, Next.js, Node) since that's
the primary stack it's built for. `security.md`, `testing.md`, `accessibility-design.md`,
`performance.md`, and `review-mode.md` are largely stack-agnostic principles and apply
regardless of language — only `stack-defaults.md` and parts of `architecture-future-proofing.md`
are JS/TS-specific.

# Data Integrity & Concurrency

The class of bug that doesn't show up in a demo, and doesn't show up in a review that only
reads one request at a time. It shows up the first time two requests land close together,
or the same request gets sent twice. A lot of AI-generated code that "looks done" quietly
isn't, specifically here.

## Race conditions (check-then-act)

The tell: "check if a row exists, then insert or update it" as two separate steps. Between
the check and the write, another request can slip in and do the same thing.
The fix: let the database enforce it — a unique constraint, an upsert
(`ON CONFLICT DO ...`), or a single atomic conditional update — instead of a check followed
by a separate write.

## Idempotency for anything that must not happen twice

Payments, order placement, sending an email, provisioning a resource: a network retry, a
double-click, or a resubmitted form must not repeat the action.
- Disabling the submit button in the UI is a UX nicety, not protection — it does nothing
  against a retried network request.
- Use an idempotency key (client-generated, checked and stored server-side) so a repeated
  request with the same key returns the original result instead of repeating the action.
  Enforce it with a unique constraint at the database level, not just an in-memory check.

## Multi-step writes

Anything that touches more than one table/collection as a single logical operation
(create an order *and* decrement stock *and* charge a card) needs a transaction, or a
saga/outbox pattern if the steps span systems that can't share one. A crash mid-operation
leaving a partial write behind is a real production bug category, not a theoretical one.

## Concurrent edits to the same record

Two users editing the same record at once: last-write-wins silently discards one person's
change. Where that matters, use optimistic concurrency — a version column or timestamp
checked on write — so a stale update fails loudly instead of clobbering someone else's data
without anyone noticing.

## The actual rule

Before writing anything that mutates state, ask: "what happens if this exact request
arrives twice, or two of these arrive at the same instant?" If the honest answer is "it
breaks," fix that as part of the implementation — not as a follow-up ticket that never gets
filed.

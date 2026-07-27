# Review Mode — Auditing Existing (Especially AI-Generated) Code

Use this whenever the user asks to review, audit, clean up, or "make production-ready" an
existing file or codebase — including code generated earlier in the same conversation.

## Process

1. Scan for Non-Negotiable violations first (see `SKILL.md`) — call these out regardless
   of anything else, even if the user only asked about one specific thing.
2. Look specifically for checklist theater, not just missing checklist items — see
   `references/loopholes.md`. A try/catch that swallows and logs, a validator that only
   checks type, a rate limiter set too loose to matter, tests that can't actually fail —
   these look done on a shallow read and need to be called out as if they weren't done at
   all, because they aren't.
3. If this code (or an earlier draft of it) was generated earlier in this same
   conversation, apply *at least* as much scrutiny as you would to a stranger's code —
   agreeing with your own prior output is the easiest way to miss something real.
4. Work through whichever reference files match what the code actually is — an API pulls
   in `security.md` + `testing.md`; a UI pulls in `accessibility-design.md` +
   `code-quality.md`; and so on.
5. Prioritize findings. Don't produce forty equally-weighted nitpicks. Group into:
   - **Critical** — security holes, data-loss risk, broken auth/authorization. Fix before
     anything else.
   - **Should fix** — real bugs, missing error handling, missing tests on critical paths,
     genuine maintainability problems.
   - **Worth considering** — style, minor performance, nice-to-haves.
6. For each finding, say what's wrong, why it matters *concretely* ("an attacker could…" /
   "this breaks when…" — not just "this is bad practice"), and the fix. Show the fix, don't
   just gesture at it.
7. Don't silently rewrite the whole file and hand back a diff with no explanation. The
   point of review mode is that the person understands what was wrong and why, not just
   that the code changed underneath them.

## Tone

Blunt and specific — not harsh for its own sake, not softened into vagueness either. Aim
for "what a good senior engineer says in a real code review," not a lecture and not empty
praise.

## Output shape (adapt to context, but roughly)

- One-line overall verdict, e.g. "Works, but has two real security gaps and no tests — not
  ready to ship as-is."
- Critical / Should fix / Worth considering, grouped as above.
- If asked to also fix it: fix Critical and Should-fix items by default. Ask before doing a
  large structural rewrite driven only by Worth-considering items.

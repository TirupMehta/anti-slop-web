# Known Loopholes (And How This Skill Closes Them)

This skill only works if the model applying it can't rationalize its way around it. Every
entry below is a real way "produce quality code" quietly degrades back into slop while
technically appearing to follow the instructions above. Read it like a threat model
pointed at this skill itself, not at the code it produces.

## Scope-shrinking loopholes

**"This is just a prototype/demo" (unearned).**
The tell: labeling something low-stakes to justify skipping the Non-Negotiables, when the
user never actually said it was throwaway.
The fix: only relax rigor below the Non-Negotiables floor when the *user* explicitly says
this is a prototype, demo, or one-off. Absent that, default to treating it like it might
handle real users and real data. Secrets, injection, password hashing, authorization, and
leaked errors never relax regardless of stated stakes — even an admitted prototype doesn't
get a plaintext password.

**"Not in the routing table, so nothing applies."**
The tell: a task that doesn't cleanly map to a row in `SKILL.md`'s table — a scraper, a
webhook receiver, a cron job — gets treated as exempt from every reference file.
The fix: the table is illustrative, not exhaustive. When a task doesn't clearly match a
row, default to `security.md` and `code-quality.md` — the two files that apply to nearly
anything that runs and touches external input.

**TODO-stubbing.**
The tell: `// TODO: add validation`, `// add rate limiting later` — a comment describing
the missing work instead of doing it, presented as if the task were complete.
The fix: a TODO comment never satisfies a Non-Negotiable. If something genuinely can't be
done right now (missing infrastructure, or the user explicitly asked to defer it), say so
out loud in the response. Don't bury it in a code comment where it reads as handled.

## Checklist-theater loopholes

Satisfying the letter of a rule without the substance behind it. Concrete examples:

- A `catch` block that only logs and swallows the error — technically "handled," not
  actually handled. Nothing decided whether to retry, surface a clean error, or fail the
  operation.
- A validator that checks `typeof x === 'string'` and calls it "input validation" for a
  field that actually needs length limits, format, and business-rule checks.
- A rate limiter applied globally at a loose threshold (e.g. 1000 req/min per IP), while
  the login endpoint — the one that actually needed 5 attempts/15 minutes — gets the same
  loose limit as everything else and is effectively unprotected.
- A comment stating `// sanitized above` next to code that doesn't actually sanitize
  anything.
- Tests that assert trivially true things, or mock away the exact logic under test, existing
  only to make a "has tests" checkbox true. If you can't describe a real bug a test would
  catch, it doesn't count toward the testing bar.

The fix, in general: when checking off a Non-Negotiable or a reference-file item, ask
"would this survive an actual attacker or an actual edge case," not "does something with
this name exist in the code."

## Confidence loopholes

**Claiming more than was verified.**
The tell: telling the user "✅ production-ready, fully secure and tested" when what
actually happened is "I wrote code that follows the patterns and didn't run it, or ran it
once on the happy path."
The fix: never present a confidence-signaling summary (checkmarks, "production-ready,"
"fully secure") unless every specific claim in it is true. State plainly what was actually
done and, just as clearly, what wasn't. "I added input validation and authorization checks;
I haven't run this against a real database, and there's no test coverage yet" is worth more
than an unearned checkmark.

**Asserting a dependency is "maintained" without checking.**
The tell: repeating `stack-defaults.md`'s recommendations indefinitely, or stating a
library is actively maintained without actually checking.
The fix: if there's a way to verify (search, package registry, changelog), use it before a
new project locks in a dependency. If that's not available in the current environment, say
so — "recommending this based on general knowledge, not a live check" — rather than stating
it as verified fact.

## Multi-turn / context loopholes

**Incremental feature creep past the checklist.**
The tell: a login form in turn one, a backend in turn two, a database in turn three — each
individual message looks small enough to skip a full audit, but the cumulative result is a
complete auth-plus-data system that never got checked as a whole.
The fix: any addition that touches auth, money, or user data triggers a check against the
*entire current state* of that feature, not just the diff being added this turn.

**Copying the existing bugs in the room.**
The tell: an existing codebase already has a SQL-concatenation pattern or a missing
authorization check, and a new feature gets added next to it "for consistency," without
fixing or even flagging the existing problem.
The fix: match the existing code's naming and formatting conventions. Never match its
Non-Negotiable violations. Flag them, even when the user only asked for something small
nearby.

**Instruction dilution over a long session.**
The tell: rigor is visibly higher in the first code block of a conversation than the fifth,
as earlier instructions fade from active attention.
The fix: re-run the Non-Negotiables self-check (`SKILL.md` Step 3) on every single
code-producing response, not just the first one in a session.

## Review-mode loopholes

**Grading your own homework leniently.**
The tell: reviewing code this same conversation generated earlier gets a softer pass than a
stranger's code would — agreeing with your own prior output is the path of least
resistance.
The fix: apply review mode with equal or greater scrutiny to your own prior output. A bug
hides best in code you already believe you got right.

**Softening findings to avoid friction.**
The tell: burying a Critical finding in "Worth considering," or hedging it so heavily it
reads as optional, because a blunt Critical finding feels confrontational.
The fix: severity is determined by actual risk, not by how the user will feel reading it. A
Critical finding stays Critical even when it's the only thing wrong with code the user is
proud of.

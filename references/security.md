# Security Baseline

Applies regardless of stack. Treat everything here as a floor for anything with a real
user, even a single one.

## Think like an attacker before you write a line

For anything touching auth, money, or another user's data, spend one sentence per
plausible misuse *before* writing the implementation — this is what actually produces
secure code, not just code that happens to pass the checklist below. A few sentences is
enough: "what if someone passes another user's ID here," "what if this request is sent
twice," "what if the payload is 50MB instead of 50 bytes," "what if this runs for a user
who was just deleted." Then make sure the code actually addresses each one named — don't
let it stay a thought exercise. `examples/secure-api-endpoint.md` shows this done in full.

## Authentication & sessions

- Hash passwords with argon2id, or bcrypt with cost ≥ 12. Never MD5/SHA1, never plaintext,
  never reversible encryption for a password.
- Use library-managed sessions/JWTs (see `stack-defaults.md`) — don't hand-write token
  signing or verification.
- Cookies are `httpOnly`, `secure`, and `sameSite=lax` (or `strict` where it won't break
  OAuth redirects).
- Rotate or invalidate sessions on password change and logout. A "logout" that doesn't
  invalidate server-side state isn't actually logout.

## Authorization

- Check "is this specific user allowed to act on this specific resource," not just "is
  someone logged in." The classic AI-slop bug: an endpoint checks auth but lets any
  logged-in user edit any other user's data because the resource ID comes straight from
  the URL with no ownership check.
- Enforce authorization server-side even if the UI already hides the button — a hidden
  button is not a security control.

## Input handling

- Validate every external input server-side against a schema (Zod or equivalent): type,
  length, format, range.
- Encode output for the context it lands in — HTML-escape for HTML, parameterize for SQL,
  escape for shell. Never build a query or command by string concatenation.
- Treat file uploads as hostile: validate type by content (not just the extension), cap
  size, store outside the web root or in object storage, never execute uploaded content.

## Secrets

- All secrets live in environment variables or a secrets manager — never in source, never
  in a client-side bundle, never in a config file that gets committed.
- `.env` goes in `.gitignore` from the first commit. Ship a `.env.example` with dummy
  values instead of the real one.

## Transport & headers

- HTTPS everywhere, HSTS enabled.
- Explicit CORS allow-list — never `*` on anything touching auth or user data.
- Set a real Content-Security-Policy, `X-Content-Type-Options: nosniff`, and
  `X-Frame-Options` (or `frame-ancestors` in the CSP). Modern frameworks make this a few
  lines of config — there's no excuse to leave it at whatever the framework defaults to
  without checking what that default actually is.

## Rate limiting & abuse

- Any public endpoint that's expensive or sensitive — login, search, email-send,
  file-processing, password reset — gets rate limiting, not just the login form.
- Lock out or throttle after repeated authentication failures.

## Dependencies

- Run a vulnerability scan (`npm audit` or a supply-chain scanner) before shipping. Treat
  high/critical results as blocking, not background noise.
- Don't add a dependency for something 20 lines of code would solve — but also don't
  hand-roll crypto, auth, or sanitization just to avoid a dependency. That trade always
  goes the wrong way.

## Data

- Principle of least privilege on database credentials — the app's DB user shouldn't be
  able to drop tables if it only ever needs to read and write rows.
- Never log sensitive data — passwords, tokens, full card numbers, anything you wouldn't
  want in a support ticket — not even at debug level.

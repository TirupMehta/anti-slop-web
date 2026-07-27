# Example: Review Mode, Before/After

A short illustration of the tone and format `review-mode.md` asks for — a small snippet, not
a real endpoint, just enough to show the shape of the output.

## The code being reviewed

```javascript
app.post("/login", async (req, res) => {
  const user = await db.query(
    `SELECT * FROM users WHERE email = '${req.body.email}'`
  );
  if (user && user.password === req.body.password) {
    res.json({ token: jwt.sign({ id: user.id }, "mysecret123") });
  } else {
    res.status(500).send(err.stack);
  }
});
```

## What review mode should produce

> **Verdict:** Doesn't work as a login route yet — there's a SQL injection hole, passwords
> are compared in plaintext, and the signing secret is hardcoded. Needs fixing before this
> touches anything real.
>
> **Critical**
> - SQL injection via string-interpolated email. An email like `' OR '1'='1` returns every
>   row in the table. Fix: a parameterized query —
>   `db.query('SELECT * FROM users WHERE email = $1', [req.body.email])`.
> - Password compared as plaintext (`user.password === req.body.password`), which means
>   passwords are being stored as plaintext too. Fix: hash with argon2id at signup, and
>   compare with `argon2.verify(user.passwordHash, req.body.password)` here.
> - JWT secret hardcoded in source. Anyone with repo access can forge a token for any user.
>   Fix: move it to an environment variable and rotate it.
>
> **Should fix**
> - The failure branch references an undefined `err` and would crash instead of returning a
>   clean 401 — and even once fixed, it shouldn't send `err.stack` to the client. Fix:
>   return a generic `401 { error: { code: "INVALID_CREDENTIALS" } }` for any login
>   failure, and log the real error server-side only.
> - No rate limiting — this endpoint can be brute-forced with no friction. Fix: see
>   `security.md`'s rate-limiting section for sensitive-endpoint limits.
>
> **Worth considering**
> - No test coverage for the failure paths (wrong password, unknown email, malformed body).

## What to notice

Three genuinely different-severity issues aren't flattened into one undifferentiated list.
Each has a concrete "an attacker could…" reason, not just "this is bad practice," and the
fix is shown inline rather than gestured at.

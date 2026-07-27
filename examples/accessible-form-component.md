# Example: An Accessible Form Component With Real States

A signup form demonstrating validation, accessibility, and the loading/error/success
states AI-generated UIs tend to skip.

## `components/SignupForm.tsx`

```tsx
"use client";

import { useState } from "react";
import { z } from "zod";

const schema = z.object({
  email: z.string().email("Enter a valid email address."),
  password: z.string().min(12, "Password must be at least 12 characters."),
});

type FieldErrors = Partial<Record<keyof z.infer<typeof schema>, string>>;

export function SignupForm() {
  const [values, setValues] = useState({ email: "", password: "" });
  const [errors, setErrors] = useState<FieldErrors>({});
  const [status, setStatus] = useState<"idle" | "submitting" | "error">("idle");
  const [serverError, setServerError] = useState<string | null>(null);

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();

    const parsed = schema.safeParse(values);
    if (!parsed.success) {
      setErrors(parsed.error.flatten().fieldErrors as FieldErrors);
      return;
    }
    setErrors({});
    setStatus("submitting");
    setServerError(null);

    try {
      const res = await fetch("/api/signup", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(parsed.data),
      });

      if (!res.ok) {
        const body = await res.json();
        setServerError(
          body.error?.code === "EMAIL_TAKEN"
            ? "An account with this email already exists."
            : "Something went wrong. Please try again."
        );
        setStatus("error");
        return;
      }

      window.location.href = "/welcome";
    } catch {
      setServerError("Couldn't reach the server. Check your connection and try again.");
      setStatus("error");
    }
  }

  return (
    <form onSubmit={handleSubmit} noValidate aria-describedby={serverError ? "form-error" : undefined}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          value={values.email}
          onChange={(e) => setValues((v) => ({ ...v, email: e.target.value }))}
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? "email-error" : undefined}
        />
        {errors.email && (
          <p id="email-error" role="alert">
            {errors.email}
          </p>
        )}
      </div>

      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          value={values.password}
          onChange={(e) => setValues((v) => ({ ...v, password: e.target.value }))}
          aria-invalid={!!errors.password}
          aria-describedby={errors.password ? "password-error" : undefined}
        />
        {errors.password && (
          <p id="password-error" role="alert">
            {errors.password}
          </p>
        )}
      </div>

      {serverError && (
        <p id="form-error" role="alert">
          {serverError}
        </p>
      )}

      <button type="submit" disabled={status === "submitting"}>
        {status === "submitting" ? "Creating account…" : "Create account"}
      </button>
    </form>
  );
}
```

## What to notice

- Validation errors are tied to their field with `aria-describedby` and announced with
  `role="alert"` — a screen reader user gets the same information a sighted user does, at
  the same moment.
- Three real states are handled: submitting (button disabled and its label changes, so a
  double-click can't fire two requests — see `data-integrity.md`), a distinguishable server
  error vs. a generic one (never exposing what the backend actually said), and success.
- Validation happens client-side for instant feedback *and* `/api/signup` still validates
  server-side — the client check is UX, never the security boundary.
- No `any`, no unused state, no comment explaining that `setStatus("submitting")` sets the
  status to submitting.

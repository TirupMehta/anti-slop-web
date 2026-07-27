# Boot-Time Environment Validation Baseline

Relying on unchecked runtime access to `process.env.VAR` leads to silent failures, runtime crashes deep inside user flows, and accidental exposure of server secrets to client bundles.

---

## 1. Boot-Time Schema Enforcement

All environment variables must be validated against a strict Zod schema during app initialization. If a required secret is missing or malformed, the app must **fail immediately at boot time** with a clear error message.

### Recommended Setup (`@t3-oss/env-nextjs` or `@t3-oss/env-core`)

```typescript
// src/env.ts
import { createEnv } from "@t3-oss/env-nextjs";
import { z } from "zod";

export const env = createEnv({
  server: {
    DATABASE_URL: z.string().url(),
    NODE_ENV: z.enum(["development", "test", "production"]).default("development"),
    STRIPE_SECRET_KEY: z.string().min(1),
  },
  client: {
    NEXT_PUBLIC_APP_URL: z.string().url(),
  },
  // Runtime binding ensures process.env is destructured safely
  experimental__runtimeEnv: {
    NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
  },
});
```

---

## 2. Server vs Client Environment Separation

* **Server-Only Secrets**: Database connection strings, API private keys (`STRIPE_SECRET_KEY`, `OPENAI_API_KEY`), and session encryption secrets must never be accessible on the client.
* **Strict Type Safety**: Access environment variables exclusively through the validated `env` object (`import { env } from "@/env"`). Do not scatter raw `process.env` references throughout code.

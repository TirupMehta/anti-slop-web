# Next.js Server Actions & React Server Components (RSC) Baseline

React Server Components and Next.js Server Actions introduce unique security and architecture pitfalls. Because Server Actions feel like plain JavaScript functions, AI code generators routinely misinterpret their execution context and introduce critical vulnerabilities.

---

## 1. Server Actions are Public HTTP Endpoints

**Critical Rule**: Every file containing `"use server"` or function marked with `"use server"` exposes a public HTTP POST endpoint accessible by anyone on the internet, regardless of whether a UI button exists.

* **Never trust input parameters**: Validate arguments with Zod inside the Server Action body, even if the caller is a typed TypeScript component.
* **Never skip authorization**: Verify session identity AND resource ownership inside every action.
* **Never rely on hidden UI controls**: A disabled button in a Client Component is not a security boundary.

```typescript
// ❌ UNSECURE: Server Action trusting arguments & missing ownership check
"use server";

export async function deleteDocument(documentId: string) {
  // Exploit: Any authenticated user can call deleteDocument("doc_123") via fetch()
  await db.delete(documents).where(eq(documents.id, documentId));
}

// ✅ PRODUCTION-GRADE: Input schema parsing + resource authorization
"use server";

import { z } from "zod";
import { auth } from "@/lib/auth";

const deleteSchema = z.object({ documentId: z.string().min(1) });

export async function deleteDocument(rawInput: unknown) {
  const session = await auth();
  if (!session?.user?.id) throw new Error("Unauthorized");

  const { documentId } = deleteSchema.parse(rawInput);

  // Enforce resource ownership at the query level
  const deleted = await db
    .delete(documents)
    .where(and(eq(documents.id, documentId), eq(documents.userId, session.user.id)))
    .returning();

  if (deleted.length === 0) {
    throw new Error("Document not found or permission denied");
  }

  return { success: true };
}
```

---

## 2. Preventing RSC Data Leaks (RSC Payload Over-fetching)

When passing data from a React Server Component to a Client Component (`"use client"`), Next.js serializes the entire object into the HTML stream (`__NEXT_DATA__` or RSC payload).

* **Never pass raw database objects to Client Components**: Returning `select *` from Prisma/Drizzle passes hidden fields (`passwordHash`, `stripeCustomerId`, `internalNotes`, `deletedAt`) to the client's browser tools.
* **Use explicit Data Transfer Objects (DTOs)**: Map database records to explicit public interfaces before passing them across the server-client boundary.

```typescript
// ❌ SLOP: Passing full DB row to client component leaks internal fields
export default async function UserProfilePage({ params }: { params: { id: string } }) {
  const userRow = await db.query.users.findFirst({ where: eq(users.id, params.id) });
  return <ClientUserProfile user={userRow} />; // Leaks userRow.passwordHash in RSC payload
}

// ✅ PRODUCTION-GRADE: Explicit DTO projection
export default async function UserProfilePage({ params }: { params: { id: string } }) {
  const userRow = await db.query.users.findFirst({ where: eq(users.id, params.id) });
  if (!userRow) notFound();

  const publicUser = {
    id: userRow.id,
    name: userRow.name,
    avatarUrl: userRow.avatarUrl,
  };

  return <ClientUserProfile user={publicUser} />;
}
```

---

## 3. Server-Only Module Isolation

To prevent server-only logic (database drivers, private keys, secrets manager SDKs) from being accidentally imported into client bundles during refactoring, enforce the `server-only` package in all backend utility modules.

```typescript
// lib/db.ts
import "server-only";
import { drizzle } from "drizzle-orm/node-postgres";

export const db = drizzle(process.env.DATABASE_URL!);
```

If a client component accidentally imports `lib/db.ts`, the build will fail immediately instead of silently bundling private code into the client JavaScript assets.

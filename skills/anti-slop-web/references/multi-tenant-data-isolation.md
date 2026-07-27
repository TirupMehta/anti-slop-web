# Multi-Tenant Data Isolation & Serverless Connection Baseline

Multi-tenant SaaS applications and serverless database deployments require strict data boundary enforcement to prevent cross-tenant data leaks and database connection exhaustion.

---

## 1. Mandatory Multi-Tenant Query Scoping

In multi-tenant systems (where users belong to Organizations, Workspaces, or Teams), every single database read, update, delete, and list operation must explicitly scope queries by `tenantId` / `workspaceId`.

### The Sub-Query Isolation Leak (Slop Trap)
A common AI error occurs in nested queries: verifying `workspaceId` on the parent object, but fetching nested child records using only `childId`.

```typescript
// ❌ SLOP: Fetching task by taskId alone allows tenant A to read tenant B's tasks
export async function getTask(workspaceId: string, taskId: string) {
  return await db.query.tasks.findFirst({
    where: eq(tasks.id, taskId), // DANGER: Missing workspaceId check!
  });
}

// ✅ PRODUCTION-GRADE: Dual-scoped query
export async function getTask(workspaceId: string, taskId: string) {
  const task = await db.query.tasks.findFirst({
    where: and(eq(tasks.id, taskId), eq(tasks.workspaceId, workspaceId)),
  });
  if (!task) throw new Error("Task not found");
  return task;
}
```

---

## 2. Soft-Delete Consistency

When schemas implement soft-deletion (`deletedAt: timestamp`), list, count, and relationship queries must consistently filter out soft-deleted records.

* **List & Count Queries**: Always include `.where(isNull(table.deletedAt))` in repository methods.
* **Unique Constraints**: Be aware that soft-deleted rows can violate database `UNIQUE` constraints (e.g., unique email or slug) unless partial unique indexes are used (`CREATE UNIQUE INDEX ON users (email) WHERE deleted_at IS NULL;`).

---

## 3. Serverless Database Connection Pooling

Serverless runtimes (Vercel Functions, AWS Lambda, Cloudflare Workers) create a new execution context on cold starts. Instantiating unpooled database client drivers inside serverless route handlers exhausts database connection limits instantly under concurrency.

### Serverless Rules
* **Global Singleton Connection**: In Node.js serverless runtimes, store the database client instance on the `globalThis` object during development to prevent connection duplication across hot-reloads.
* **Use HTTP / WebSockets Proxy Poolers**: For PostgreSQL on serverless, use connection poolers (Neon Serverless Driver, Supabase Connection Pooler / Transaction Mode, Prisma Accelerate, or AWS RDS Proxy).

```typescript
// lib/db.ts - Global singleton pattern for serverless ORM instances
import { drizzle } from "drizzle-orm/neon-http";
import { neon } from "@neondatabase/serverless";

const globalForDb = globalThis as unknown as { conn: ReturnType<typeof neon> | undefined };

const sql = globalForDb.conn ?? neon(process.env.DATABASE_URL!);
if (process.env.NODE_ENV !== "production") globalForDb.conn = sql;

export const db = drizzle(sql);
```

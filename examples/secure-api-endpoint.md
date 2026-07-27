# Example: A Production-Grade API Endpoint

A fully worked example of one endpoint — placing an order — applying every pillar in this
skill at once. It's deliberately narrow in scope (one endpoint) so the whole thing is
readable, not because a real app only needs this much code.

Read this once before generating something structurally similar for the first time in a
session; it's a faster calibration than re-reading every reference file's prose.

## `app/api/orders/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";
import { getSession } from "@/lib/auth"; // Better Auth session helper
import { db } from "@/lib/db";
import { orders, orderItems, inventory } from "@/lib/db/schema";
import { rateLimit } from "@/lib/rate-limit";
import { logger } from "@/lib/logger";
import { sql } from "drizzle-orm";

// Threat model for this endpoint, one line each (see security.md):
// - Someone submits an order for another user   -> userId comes from the session, never the body.
// - Someone double-submits (retry, double-click) -> idempotencyKey makes a repeat a no-op.
// - Someone orders more stock than exists         -> stock check + decrement happen in one transaction.
// - Someone hammers this endpoint                 -> rate-limited per user.

const placeOrderSchema = z.object({
  idempotencyKey: z.string().uuid(),
  items: z
    .array(
      z.object({
        productId: z.string().uuid(),
        quantity: z.number().int().positive().max(100),
      })
    )
    .min(1)
    .max(50),
});

export async function POST(req: NextRequest) {
  const session = await getSession(req);
  if (!session) {
    return NextResponse.json({ error: { code: "UNAUTHENTICATED" } }, { status: 401 });
  }

  const { success: withinLimit } = await rateLimit.check(`orders:${session.userId}`);
  if (!withinLimit) {
    return NextResponse.json({ error: { code: "RATE_LIMITED" } }, { status: 429 });
  }

  const parsed = placeOrderSchema.safeParse(await req.json().catch(() => null));
  if (!parsed.success) {
    return NextResponse.json(
      { error: { code: "INVALID_INPUT", details: parsed.error.flatten() } },
      { status: 400 }
    );
  }
  const { idempotencyKey, items } = parsed.data;

  // A unique constraint on (userId, idempotencyKey) makes a retried request a no-op
  // instead of a second order — see data-integrity.md.
  const existing = await db.query.orders.findFirst({
    where: (o, { and, eq }) =>
      and(eq(o.userId, session.userId), eq(o.idempotencyKey, idempotencyKey)),
  });
  if (existing) {
    return NextResponse.json({ orderId: existing.id }, { status: 200 });
  }

  try {
    const orderId = await db.transaction(async (tx) => {
      for (const item of items) {
        // Stock check and decrement happen as one atomic conditional update inside the
        // transaction, not a separate "check then write" — see data-integrity.md.
        const result = await tx
          .update(inventory)
          .set({ stock: sql`${inventory.stock} - ${item.quantity}` })
          .where(
            sql`${inventory.productId} = ${item.productId} AND ${inventory.stock} >= ${item.quantity}`
          );

        if (result.rowCount === 0) {
          throw new Error(`INSUFFICIENT_STOCK:${item.productId}`);
        }
      }

      const [order] = await tx
        .insert(orders)
        .values({ userId: session.userId, idempotencyKey })
        .returning({ id: orders.id });

      await tx.insert(orderItems).values(
        items.map((item) => ({
          orderId: order.id,
          productId: item.productId,
          quantity: item.quantity,
        }))
      );

      return order.id;
    });

    return NextResponse.json({ orderId }, { status: 201 });
  } catch (err) {
    if (err instanceof Error && err.message.startsWith("INSUFFICIENT_STOCK:")) {
      const productId = err.message.split(":")[1];
      return NextResponse.json({ error: { code: "INSUFFICIENT_STOCK", productId } }, { status: 409 });
    }

    // Unexpected failure: log with enough context to debug it, tell the user nothing
    // internal. Never leak err.message or a stack trace to the client here.
    logger.error("order_placement_failed", { userId: session.userId, err });
    return NextResponse.json({ error: { code: "INTERNAL_ERROR" } }, { status: 500 });
  }
}
```

## `app/api/orders/route.test.ts`

```typescript
import { describe, it, expect, vi } from "vitest";
import { POST } from "./route";

// These tests target the behavior that actually matters for this endpoint — not
// "does it return 200," but the specific failure modes named in the threat-model
// comment above. Each one is a bug this endpoint could plausibly ship with.

describe("POST /api/orders", () => {
  it("rejects a request with no session", async () => {
    const res = await POST(makeRequest({ items: [{ productId: VALID_ID, quantity: 1 }] }, { authed: false }));
    expect(res.status).toBe(401);
  });

  it("returns the original order on a repeated idempotency key instead of creating a second one", async () => {
    const body = { idempotencyKey: SAME_KEY, items: [{ productId: VALID_ID, quantity: 1 }] };
    const first = await POST(makeRequest(body));
    const second = await POST(makeRequest(body));

    expect((await second.json()).orderId).toBe((await first.json()).orderId);
    expect(await countOrdersForKey(SAME_KEY)).toBe(1);
  });

  it("rejects an order that exceeds available stock without partially decrementing it", async () => {
    const res = await POST(
      makeRequest({ idempotencyKey: crypto.randomUUID(), items: [{ productId: LOW_STOCK_ID, quantity: 9999 }] })
    );
    expect(res.status).toBe(409);
    expect(await getStock(LOW_STOCK_ID)).toBe(ORIGINAL_STOCK); // unchanged, not partially decremented
  });

  it("never includes the raw error message in a 500 response", async () => {
    vi.spyOn(dbModule, "transaction").mockRejectedValueOnce(new Error("connection refused at 10.0.0.4:5432"));
    const res = await POST(
      makeRequest({ idempotencyKey: crypto.randomUUID(), items: [{ productId: VALID_ID, quantity: 1 }] })
    );
    expect(JSON.stringify(await res.json())).not.toContain("10.0.0.4");
  });
});
```

## What to notice

- The threat model is written down as a comment, then every line of it is actually
  addressed in the code below it — not left as an aspiration.
- Every error response has the same shape (`{ error: { code, ... } }`), and none of them
  leak internals.
- The idempotency and stock-check tests aren't testing "does the happy path work" — they're
  testing the exact failure modes named up front. That's the difference between tests that
  exist and tests that actually catch something.
- There isn't a single filler comment explaining what a line already says; the comments
  that do exist explain *why* (the threat model, the reasoning behind the transaction).

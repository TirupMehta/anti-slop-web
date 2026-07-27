# anti-slop-web

An AI Agent Skill that enforces production-grade security, data integrity, and software engineering standards when generating or reviewing web applications.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## Overview

AI code generators produce code that runs, but frequently omit critical production controls that senior engineers apply by default:

* **Resource-level authorization checks**: Verifying authentication while omitting user ownership validation (IDOR/BOLA).
* **Superficial validation**: Catch blocks that swallow errors, unchecked type assertions, and missing server-side schema parsing.
* **Database and state vulnerabilities**: String-built queries, missing transaction boundaries, and unhandled race conditions on state mutations (e.g., duplicate payments or cancellations).
* **Leaky error handling**: Returning internal database traces to the client UI.

`anti-slop-web` is a structured instruction set compatible with Claude Code, Cursor, Windsurf, and Gemini/Antigravity agents. It forces agents to audit every generated endpoint, component, and query against senior engineering standards before marking tasks complete.

---

## Installation

### Claude Code CLI & Agent Frameworks
Clone into your project's `.agents/skills` directory:

```bash
mkdir -p .agents/skills
git clone https://github.com/TirupMehta/anti-slop-web.git .agents/skills/anti-slop-web
```

Or copy globally to `~/.gemini/config/skills/anti-slop-web/`.

### Cursor
Copy `SKILL.md` to your workspace `.cursor/rules/`:

```bash
mkdir -p .cursor/rules
cp SKILL.md .cursor/rules/anti-slop-web.mdc
```

### Windsurf & Other Agents
Reference `SKILL.md` directly in your workspace instructions (`.windsurfrules` or `AGENTS.md`).

---

## Code Generation Baseline: Before and After

### Default AI Output
```typescript
// Unsecure: Missing authorization check, missing input validation, unhandled errors
app.post("/api/orders/:id/cancel", async (req, res) => {
  const order = await db.query(`SELECT * FROM orders WHERE id = '${req.params.id}'`);
  await db.query(`UPDATE orders SET status = 'cancelled' WHERE id = '${req.params.id}'`);
  res.json({ success: true });
});
```

### Enforced by `anti-slop-web`
```typescript
// Enforced: Schema validation, resource ownership, atomic update, error masking
app.post("/api/orders/:id/cancel", async (req, res) => {
  try {
    const { id: orderId } = cancelOrderSchema.parse(req.params);
    const userId = req.user?.id;
    if (!userId) return res.status(401).json({ error: "Unauthorized" });

    const updatedCount = await db.orders.updateMany({
      where: { id: orderId, userId, status: { not: "cancelled" } },
      data: { status: "cancelled", cancelledAt: new Date() },
    });

    if (updatedCount === 0) {
      return res.status(404).json({ error: "Order not found or already processed" });
    }

    return res.status(200).json({ success: true });
  } catch (error) {
    logger.error("Order cancellation failed", { error, orderId: req.params.id });
    return res.status(500).json({ error: "Internal server error" });
  }
});
```

---

## Core Non-Negotiables

Every piece of code touched by this skill must satisfy these baseline requirements:

1. **Zero hardcoded secrets**: All credentials, keys, and tokens must use environment variables.
2. **Parameterized database access**: No string-concatenated SQL queries under any circumstance.
3. **Password hashing**: Use `argon2id` or `bcrypt` (cost ≥ 12).
4. **Server-side validation**: All external inputs must be validated against a schema (e.g., Zod) on the server.
5. **Masked stack traces**: Internal paths and raw database errors must never reach client responses.
6. **Explicit authorization boundaries**: Every mutating route must check resource ownership, not just authentication status.
7. **Maintained dependencies**: Do not introduce deprecated or abandoned packages.
8. **Strict TypeScript**: No unannotated `any` types.
9. **Handled async promises**: No silent promise swallows in catch blocks.
10. **Complete UI states**: Data-fetching components must explicitly handle loading, empty, and error states.
11. **Explicit security headers & CORS**: Security headers and CORS origins must be configured deliberately.
12. **Critical path testing**: Core business logic must include verification tests.
13. **Honest gap reporting**: Agents must state unverified gaps explicitly rather than asserting unearned production readiness.
14. **Data integrity**: State mutations (payments, orders) must be protected against duplicate execution at the data layer.

---

## Reference Architecture

The primary [`SKILL.md`](SKILL.md) routes tasks to targeted reference documents:

| Task Type | Reference Document | Primary Focus |
|---|---|---|
| Frontend UI / Components | [`accessibility-design.md`](references/accessibility-design.md), [`code-quality.md`](references/code-quality.md) | WCAG AA accessibility, state management, micro-interactions |
| Backend & APIs | [`security.md`](references/security.md), [`testing.md`](references/testing.md) | OWASP security, authorization boundaries, edge testing |
| Database & State | [`data-integrity.md`](references/data-integrity.md), [`performance.md`](references/performance.md) | Idempotency, atomic operations, query optimization |
| Audits & Code Reviews | [`review-mode.md`](references/review-mode.md), [`loopholes.md`](references/loopholes.md) | Detection of checklist rationalizations and TODO stubs |
| Project Scaffolding | [`stack-defaults.md`](references/stack-defaults.md), [`architecture-future-proofing.md`](references/architecture-future-proofing.md) | Ecosystem defaults, clean abstractions, operations |

---

## Contributing

Please review [`CONTRIBUTING.md`](CONTRIBUTING.md) for details on submitting pull requests, updating reference manuals, and reporting issues.

## License

Distributed under the [MIT License](LICENSE).

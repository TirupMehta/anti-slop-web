# anti-slop-web

> **Stop AI code slop. Enforce senior-grade security, data integrity, and engineering standards across Claude Code, Cursor, Windsurf, and AI coding agents.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Agent Skill Standard](https://img.shields.io/badge/Agent_Skill-v1.0-emerald.svg)](SKILL.md)
[![Compatible Tools](https://img.shields.io/badge/Compatible-Claude_Code_%7C_Cursor_%7C_Windsurf_%7C_Gemini-violet.svg)](#installation)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## 🛑 The Problem: AI Code Slop

LLM code generators usually produce code that runs. That is a low bar. 

Under the surface, default AI outputs consistently skip the unspoken judgments senior engineers make automatically:
* **Missing authorization checks**: Verifying that a user is logged in, but forgetting to check if user `$A` owns resource `$B` (IDOR / BOLA vulnerabilities).
* **Checklist theater**: Swallowing errors in empty catch blocks or checking `typeof x === 'string'` and calling it validated.
* **Unsafe database access**: String-concatenated queries or missing unique constraints / idempotency key protections for payments and state changes.
* **Leaky error handling**: Returning raw database stack traces to the client UI.
* **Prototype rationalization**: Quietly dropping input schemas, WCAG accessibility, security headers, and loading/error states because the prompt was "just a quick feature."

`anti-slop-web` is a modular **AI Agent Skill** designed to eliminate this drift. It forces LLM agents to clear the exact same production checklist a skeptical staff engineer runs during code review.

---

## ⚡ Quick Start / Installation

Copy or clone `anti-slop-web` into your project or global AI agent config folder:

### 1. Claude Code CLI / Antigravity / Gemini IDE
Add to your project's `.agents` customization directory:
```bash
mkdir -p .agents/skills
git clone https://github.com/TirupMehta/anti-slop-web.git .agents/skills/anti-slop-web
```
*Or copy globally to `~/.gemini/config/skills/anti-slop-web/`.*

### 2. Cursor
Add to your workspace `.cursor/rules/` or global Cursor settings:
```bash
mkdir -p .cursor/rules
cp SKILL.md .cursor/rules/anti-slop-web.mdc
```

### 3. Windsurf & Other Agentic Frameworks
Point your system prompt or `.windsurfrules` / `AGENTS.md` to reference `SKILL.md`.

---

## 🔍 Before vs. After: What This Skill Changes

### ❌ Default AI Output (Slop)
```typescript
// Typical AI-generated endpoint: Looks fine at first glance, but breaks in production
app.post("/api/orders/:id/cancel", async (req, res) => {
  const user = req.user; // Authenticated? Yes. Authorized? Unknown.
  
  // Bug 1: No check if req.params.id belongs to req.user.id (IDOR Vulnerability)
  // Bug 2: No idempotency check (Race condition / duplicate cancellation)
  const order = await db.query(`SELECT * FROM orders WHERE id = '${req.params.id}'`); // Bug 3: SQL Injection
  
  await db.query(`UPDATE orders SET status = 'cancelled' WHERE id = '${req.params.id}'`);
  res.json({ success: true }); // Bug 4: No try/catch, unhandled promise rejections leak DB stack traces
});
```

### ✅ Output Enforced by `anti-slop-web`
```typescript
// Enforced by anti-slop-web: Strict security, auth boundaries, & data integrity
app.post("/api/orders/:id/cancel", async (req, res) => {
  try {
    // 1. Server-side Input Schema Validation
    const { id: orderId } = parseParams(cancelOrderSchema, req.params);
    const userId = req.user?.id;
    if (!userId) return res.status(401).json({ error: "Unauthorized" });

    // 2. Resource-Level Authorization & Atomic Update (Data Integrity)
    // Parameterized query prevents SQLi + guarantees user ownership + enforces idempotency
    const updatedCount = await db.orders.updateMany({
      where: { id: orderId, userId: userId, status: { not: "cancelled" } },
      data: { status: "cancelled", cancelledAt: new Date() },
    });

    if (updatedCount === 0) {
      // 3. Prevent information leaks: Don't reveal if order didn't exist vs belonged to another user
      return res.status(404).json({ error: "Order not found or already processed" });
    }

    return res.status(200).json({ success: true });
  } catch (error) {
    // 4. Safe Error Masking & Structured Logging
    logger.error("Failed to cancel order", { error, orderId: req.params.id });
    return res.status(500).json({ error: "Internal server error" });
  }
});
```

---

## 🛡️ The 14 Non-Negotiables

`anti-slop-web` strictly enforces 14 core rules on **every single line of code generated**, regardless of project size:

1. **Zero Hardcoded Secrets**: Env vars only. Never commit API keys or passwords.
2. **Parameterized Queries**: Zero string-concatenated database statements.
3. **Strong Hashing**: Argon2id or bcrypt (cost ≥ 12) for passwords.
4. **Server-Side Validation**: External inputs validated via schemas; client validation is UX, not security.
5. **Masked Error Stack Traces**: Users never see raw DB errors or system paths.
6. **Resource Authorization Boundaries**: Verify user permission on *every* resource ID.
7. **Maintained Dependencies**: No abandoned or unmaintained packages added.
8. **Strict TypeScript**: No silent `any` types allowed without explicit justification.
9. **Handled Async Failures**: Zero unhandled or swallowed promises.
10. **Triple-State UI**: Data-fetching UI must handle loading, empty, and error states.
11. **Explicit Security Headers**: Headers & CORS explicitly configured and audited.
12. **Critical Path Tests**: Nothing ships without verification tests.
13. **Honest Confidence**: State unverified gaps plainly instead of claiming unearned "production-ready" status.
14. **Data Integrity & Idempotency**: State changes (payments, cancellations) protected at the data layer against race conditions.

---

## 📚 Skill Reference Architecture

The skill modularly delegates deep domain checklists to reference manuals when specific tasks are triggered:

| Task Type | Reference Manual Loaded | Focus Area |
|---|---|---|
| New Component / UI Page | [`accessibility-design.md`](references/accessibility-design.md), [`code-quality.md`](references/code-quality.md) | WCAG AA compliance, state management, micro-animations |
| API / Backend Logic | [`security.md`](references/security.md), [`testing.md`](references/testing.md) | OWASP Top 10, authorization boundaries, edge testing |
| Database & State Changes | [`data-integrity.md`](references/data-integrity.md), [`performance.md`](references/performance.md) | Idempotency keys, atomic updates, N+1 query prevention |
| Code Audit / Review Mode | [`review-mode.md`](references/review-mode.md), [`loopholes.md`](references/loopholes.md) | Catching subtle slop, rationalizations, and TODO stubs |
| Full App Scaffold | [`stack-defaults.md`](references/stack-defaults.md), [`architecture-future-proofing.md`](references/architecture-future-proofing.md) | Modern tech stack, clean abstractions, deployment ops |

---

## 🤝 Contributing

Contributions, additional security vectors, and framework updates are welcome! Please read [`CONTRIBUTING.md`](CONTRIBUTING.md) and submit a Pull Request.

## 📜 License

Licensed under the [MIT License](LICENSE).

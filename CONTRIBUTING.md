# Contributing to anti-slop-web

Thank you for helping improve `anti-slop-web`. This skill exists to enforce senior-level engineering rigor and eliminate shallow AI code slop in modern web applications.

## How to Contribute

We welcome contributions that expand security baselines, add modern framework edge cases (e.g. Next.js Server Actions, Remix, SvelteKit), or improve reference checklists.

### 1. Adding or Modifying Rules

When proposing a new rule or updating an existing reference file in `references/`:
* **Provide a concrete "why"**: Explain the actual failure mode or security exploit the rule prevents (e.g., IDOR, race conditions, serverless connection leaks).
* **Keep instructions actionable for LLMs**: Rules must be unambiguous so an AI agent can evaluate its own code against them.
* **Include Before/After code examples**: If adding a complex security check, update or add an example in `examples/`.

### 2. Submitting a Pull Request

1. Fork the repository and create a new branch for your feature or fix.
2. Verify that all relative markdown links in `README.md` and `SKILL.md` point to existing files.
3. Test the skill against real coding tasks in your IDE/CLI agent to confirm it triggers cleanly.
4. Submit a Pull Request targeting the `main` branch with a clear summary of your changes.

## Reporting Issues

If you find a broken link, an outdated stack recommendation, or an edge case where AI agents bypass a rule, please [open an Issue](https://github.com/TirupMehta/anti-slop-web/issues).

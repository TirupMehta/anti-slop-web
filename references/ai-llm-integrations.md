# AI & LLM Integrations Baseline

Integrating LLMs (OpenAI, Anthropic, Gemini, local models, RAG pipelines) introduces cost vulnerabilities, prompt injection risks, and streaming execution edge cases.

---

## 1. Secret Protection & API Key Isolation

* **Zero Client-Side LLM Keys**: Never prefix LLM API keys with public environment variable flags (`NEXT_PUBLIC_`, `VITE_`, `REACT_APP_`). All LLM SDK invocations must execute server-side (API routes, Server Actions, Node backends).
* **Verify Proxy Boundaries**: Client applications must send requests to your backend endpoints, which then securely proxy calls to provider APIs using server environment variables.

---

## 2. Cost Control & Denial-of-Wallet Protection

Unbounded LLM endpoints can be abused via automated scripts to exhaust provider quota limits or generate massive bills.

* **Mandatory Per-User Rate Limiting**: Every LLM endpoint must enforce rate limits keyed by `userId` (or IP address for unauthenticated public demos) using sliding window counters (e.g., Redis / Upstash).
* **Strict Max Token Controls**: Always pass `max_tokens` (or equivalent provider limits) in LLM generation requests. Never leave max generation tokens uncapped.
* **Input Truncation**: Enforce maximum character limits on user prompts passed to LLMs to prevent token inflation attacks.

```typescript
// ✅ PRODUCTION-GRADE: Rate limiting + max tokens + input bounding
export async function POST(req: Request) {
  const session = await auth();
  if (!session?.user?.id) return new Response("Unauthorized", { status: 401 });

  // 1. Sliding window rate limit (e.g. max 10 requests per minute per user)
  const { success } = await rateLimiter.limit(`llm_${session.user.id}`);
  if (!success) return new Response("Rate limit exceeded", { status: 429 });

  // 2. Validate & bound user input length
  const { prompt } = promptSchema.parse(await req.json());
  const boundedPrompt = prompt.slice(0, 2000); // Cap input length

  // 3. Provider invocation with hard max_tokens cap
  const response = await openai.chat.completions.create({
    model: "gpt-4o-mini",
    max_tokens: 500,
    messages: [{ role: "user", content: boundedPrompt }],
  });

  return Response.json({ text: response.choices[0].message.content });
}
```

---

## 3. Defense Against Prompt Injection

User input embedded directly into system prompts can hijack model instructions or bypass security constraints.

* **Separate System & User Roles**: Pass system instructions in the `system` role and user content strictly in the `user` role. Never interpolate user text into system prompt strings (`"You are a helpful assistant. User input: " + userInput`).
* **Output Validation**: Treat LLM output as untrusted external data. If the model returns structured JSON, validate it against a Zod schema before storing it in a database or returning it to client state.

---

## 4. Resilient Streaming UIs (SSE / ReadableStream)

When streaming LLM tokens to a frontend UI via Server-Sent Events (SSE) or Fetch Streams:

* **Signal & Abort Handling**: Listen for request abort signals (`req.signal`) on the server so LLM generation halts immediately if the client disconnects or closes the tab.
* **Client Stream Error Boundaries**: Client UI stream hooks must handle network dropouts, provider rate limits, and partial stream corruptions gracefully without crashing the UI component tree.

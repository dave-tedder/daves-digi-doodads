---
name: ai-sdk-provider-defaults
description: "Use when writing or editing TypeScript/JavaScript code that imports from @ai-sdk/openai or instantiates a language model against an OpenAI-compatible backend (OpenRouter, Groq, Together, DeepInfra, Fireworks). Guards against the silent /v1/responses vs /v1/chat/completions routing bug introduced in @ai-sdk/openai 3.x."
---

# `@ai-sdk/openai` Provider Defaults: `provider(id)` vs `provider.chat(id)`

Starting in `@ai-sdk/openai@3.x`, calling the default provider function with just a model ID returns a **Responses API** language model, not a Chat Completions one. The generated request goes to `/v1/responses` (OpenAI's new per-turn persistence-aware endpoint), not `/v1/chat/completions`.

```ts
// v3+ default — hits /v1/responses
const model = openrouter("anthropic/claude-sonnet-4.6");

// Explicit chat completions — hits /v1/chat/completions
const model = openrouter.chat("anthropic/claude-sonnet-4.6");
```

OpenRouter, Groq, Together, DeepInfra, Fireworks, and every other OpenAI-compatible aggregator implement `/chat/completions`. **None of them implement `/responses`.** When the SDK sends a Responses-API request, those backends return a 400 with a schema-mismatch error. In a streaming context the error surfaces as a generic `AI_APICallError: Invalid Responses API request` in the server logs and a 500 to the client.

## The rule

When instantiating a model against any OpenAI-compatible backend that isn't OpenAI itself, always call `.chat(...)` explicitly:

```ts
import { createOpenAI } from "@ai-sdk/openai";

const openrouter = createOpenAI({
  baseURL: "https://openrouter.ai/api/v1",
  apiKey: process.env.OPENROUTER_API_KEY,
});

const result = streamText({
  model: openrouter.chat("anthropic/claude-sonnet-4.6"),
  // ...
});
```

`.chat(id)` internally instantiates `OpenAIChatLanguageModel` with `path: "/chat/completions"`. Verify in `node_modules/@ai-sdk/openai/dist/index.mjs` by grepping for `provider.chat = createChatModel`.

## When calling OpenAI directly

The bare `openai(id)` form is fine when `baseURL` is OpenAI itself (openai.com), since OpenAI does implement Responses. Use `openai.chat(id)` if you specifically want the legacy Chat Completions contract (e.g. for compatibility with older tooling that expects the `choices[].delta.content` shape).

## Silent-failure mode

The mistake doesn't produce a loud client error. A fetch to `/v1/responses` on OpenRouter just returns 400, the SDK wraps it as `AI_APICallError`, and whatever's consuming the stream (Next.js route handler, server-side iterator, etc.) re-raises it as a 500. The frontend sees a generic streaming fetch failure. Easy to misattribute as an env var issue or runtime problem.

## Detection

Before debugging anything else on a chat-fetch 500, check:

1. The backend URL in the SDK error trace. If `url` ends in `/responses`, you have this bug.
2. Whether the provider instantiation uses `openrouter(id)` vs `openrouter.chat(id)`.

Direct curl with the exact model slug is the fastest way to confirm the model + key work independent of the SDK:

```bash
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"anthropic/claude-sonnet-4.6","messages":[{"role":"user","content":"ping"}],"max_tokens":10}'
```

If that returns 200 but the SDK-routed call still 500s with a schema error, the provider-default trap is the cause.

## Real-world use

Open Brain Session 53 (2026-04-17) spent ~30 minutes chasing this when the dashboard's `/api/chat` endpoint started returning 500 after the AI SDK was bumped to 6.x. Fix was `openrouter(id)` → `openrouter.chat(id)`. Noted in `dave-tedder/open-brain-dashboard` commit `d3f615b`.

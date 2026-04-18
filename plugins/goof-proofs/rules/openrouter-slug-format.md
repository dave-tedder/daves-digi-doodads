# OpenRouter Model Slug Format

OpenRouter model slugs use a dot as the minor-version separator, not a dash. The direct Anthropic model ID format uses a dash for the same separator. Easy to mis-type when converting between them.

## Examples

| Direct Anthropic ID | OpenRouter slug |
|---|---|
| `claude-sonnet-4-6` | `anthropic/claude-sonnet-4.6` |
| `claude-opus-4-6` | `anthropic/claude-opus-4.6` |
| `claude-haiku-4-5-20251001` | `anthropic/claude-haiku-4.5` (date stamp dropped in OpenRouter) |

The direct ID can be pasted into the Anthropic SDK (`messages.create({ model: "claude-sonnet-4-6" })`). The OpenRouter slug goes into the OpenRouter chat-completions request body (`{ model: "anthropic/claude-sonnet-4.6" }`).

## The silent-failure mode

A bad OpenRouter slug does NOT throw a loud client error in the Edge Function code paths used by brain-digest and compile-pages. The fetch completes, the response contains an error object, and the caller's try/catch logs the failure but the cron-triggered digest still posts "couldn't synthesize today" to Slack. Same shape as a model timeout. Easy to miss.

## Before editing

Verify the current slug against https://openrouter.ai/models or https://openrouter.ai/anthropic/<model-name>. The specific model page renders a 200 with "Released on..." if the slug resolves. A 404 or "model not found" page means the slug is wrong.

## Real-world use

Open Brain Session 45 (2026-04-16) executed the audit P0 — bumping `anthropic/claude-sonnet-4` to `anthropic/claude-sonnet-4.6` in two Edge Functions. Verified the slug via WebFetch on https://openrouter.ai/anthropic/claude-sonnet-4.6 before touching either file. The audit document explicitly flagged the dot-vs-dash ambiguity as "uncertain, verify before deploy."

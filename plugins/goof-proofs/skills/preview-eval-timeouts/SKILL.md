---
name: preview-eval-timeouts
description: "Use when calling `mcp__Claude_Preview__preview_eval` on slow backend work (any fetch over ~20s — Claude Sonnet endpoints, generation jobs), when seeing 'preview_eval timed out', or when recovering from a timeout on a write-mutation request. CRITICAL: always check the DB for side effects before retrying a timed-out write."
---

# `preview_eval` Has a 30s Hard Timeout

`mcp__Claude_Preview__preview_eval` times out at 30 seconds. Any asynchronous work inside the evaluated expression — especially `fetch()` calls to slow backend endpoints — will abort at 30s and return "preview_eval timed out" even if the request is still in flight server-side. Worst case: the server finishes the work (writes a row, sends an email, etc.) but the frontend never sees the response, and the eval reports failure.

## Common hit

Calling `POST /api/meal-plans/generate` or any other Claude Sonnet-backed endpoint from inside the preview. Full meal plan generation runs ~25 seconds end-to-end, which is just under the limit on a good day and over it on a slow day.

## Workaround

Pull the auth token out via a fast `preview_eval`, then make the long request via `curl` from Bash with a larger `timeout`:

```js
// Fast eval: grab the token
(() => {
  const keys = Object.keys(localStorage).filter(k => k.includes('auth-token'));
  const session = JSON.parse(localStorage.getItem(keys[0]));
  return session?.access_token;
})()
```

```bash
# Bash: the actual long request
curl -X POST http://localhost:3001/api/long-endpoint \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{...}' \
  -o /tmp/response.json -w "HTTP: %{http_code}\nTime: %{time_total}s\n"
# (Bash tool accepts timeout up to 600000ms / 10 minutes)
```

The Supabase JWT is stored at an `sb-*-auth-token` key in localStorage. Extract the `.access_token` field, pass it as a Bearer token, and curl hits the same endpoint without the 30s ceiling.

## Don't fight the timeout

If you see a timeout on `preview_eval`, don't retry the same eval hoping it's faster this time. Check the DB / server log first to see if the server-side work completed. If it did, you've just lost the response payload, not the effect. If it didn't, switch to the curl approach immediately.

## Critical: the request did NOT necessarily fail

A `preview_eval` timeout means the JavaScript Promise never resolved within 30s. It does NOT mean the underlying request failed. In the worst case, the server processed the request fully — wrote rows, spent money, triggered webhooks — and the eval just never saw the response. **Always check the DB for side effects before retrying a write-mutation request.** A write endpoint that timed out once and "looks like it failed" may have actually succeeded, and retrying it will create duplicates.

Real-world failure mode (Tedder Trainer Session 44, 2026-04-14): a `POST /api/meal-plans/generate` call from inside `preview_eval` timed out at 30s. The next session assumed the fetch hadn't landed and re-ran it via curl, which created a second plan. Then the user deleted what they thought was the duplicate (the curl one, the most recent by id) — but the preview_eval request had also completed server-side, creating an earlier plan that got missed. That phantom plan sat in the DB and silently overrode the user's active meal plan in the app UI because the frontend's `/meal-plans/current` endpoint returns the newest matching row. Confusing and alarming.

## Recovery procedure after a timeout on a write-mutation request

1. Query the DB for any row the request would have created (by timestamp, user, or idempotency key).
2. If one exists, that IS the response you didn't see. Do not retry. Either use it or clean it up.
3. If none exists, only then retry via curl.
4. When cleaning up, sort by `created_at` / id and check ALL candidates, not just "the most recent one." Two phantom rows are possible if both `preview_eval` and your retry succeeded.

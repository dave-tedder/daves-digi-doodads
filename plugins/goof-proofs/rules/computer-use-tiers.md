# Computer-Use MCP: Application Tier Awareness

Some applications are granted at "click" tier only (visible left-clicks and scrolling, no typing or key presses). This applies to apps with terminal or IDE capabilities, including the Shortcuts app.

When an app is click-tier:
- Plan for a hybrid workflow: agent clicks, user types
- Identify which steps require text input before starting
- Guide the user through typing steps conversationally while handling navigation clicks

Known click-tier apps: Shortcuts, Terminal, any IDE or code editor.

## Browser Automation: Chrome vs Brave

Claude-in-Chrome MCP controls Chrome, not Brave. For browser automation tasks (navigating sites, filling forms, extracting data), use Chrome tabs via Claude-in-Chrome MCP tools. Do not request Brave Browser access via computer-use MCP for navigation. Computer-use screenshot access to Brave is read-only and useful only for visual verification.

## Claude-in-Chrome Tab Groups and Login State

Claude-in-Chrome creates its own tab group. Tabs in this group do NOT inherit login sessions from the user's existing browser tabs. Navigating to an authenticated page (Notion, Venmo, bank portals, etc.) in a Claude-in-Chrome tab will hit a login wall.

Workarounds:
- Have the user download/screenshot what's needed from their logged-in tabs
- Have the user paste the URL into a Claude-in-Chrome tab while already logged in (cookies may carry over within the same Chrome profile)
- Use API/MCP access instead of browser automation when available (e.g., Notion MCP instead of Notion web UI)

## Computer-Use Chrome Read Tier + Hybrid Pattern

Computer-use grants Chrome at "read" tier only (visible in screenshots, no clicks or typing). For tasks that need both visual understanding AND authenticated browser interaction, use a hybrid approach:
- **Control Chrome** for navigation (open_url, execute_javascript, close_tab). Operates on existing tabs with full cookie/session access.
- **Computer-use** for reading (screenshot). Gives visual understanding of the page (images, video thumbnails, attachment types, UI elements).

Control Chrome's `get_page_content` returns text only. It cannot distinguish images, videos, or attachment types. If the task requires visual understanding of page content, use computer-use screenshots instead of text extraction.

## Control Chrome Reliability

Control Chrome's `get_page_content` and `execute_javascript` intermittently fail with "Chrome is not running" even when `open_url` works and Chrome is visible. When this happens, use Claude-in-Chrome's `read_page`/`get_page_text` for content extraction, or WebFetch for public pages. WebFetch is particularly good for scouting multiple sites in parallel (can run 8-9 concurrent fetches).

## Claude-in-Chrome: Iframe Limitations

Claude-in-Chrome's `read_page` cannot see form fields inside iframes (e.g., embedded Wufoo, Typeform, Google Forms). The accessibility tree only returns elements from the parent page. Workaround: load the iframe source URL directly in a Claude-in-Chrome tab. For Wufoo forms, the direct URL pattern is `adminfoot.wufoo.com/forms/[form-id]/` (find the form ID in the iframe src attribute or page source).

## Cowork Agent Tool Fallbacks

Cowork agents will improvise tool fallbacks unless explicitly told not to. If a Cowork prompt uses Control Chrome and it fails, the agent may silently fall back to Claude-in-Chrome (which creates isolated tabs without login sessions). Any Cowork prompt should include hard rules about which tools NOT to fall back to when the primary tool fails.

## Claude-in-Chrome: form_input vs type on claude.ai

On claude.ai pages (and likely other complex SPAs), the `type` action frequently fails with "Detached while handling command." Use `form_input` with element ref IDs from `read_page` instead. It reliably sets values on text inputs and textareas. The `type` action is fine for simple pages but unreliable on React-heavy SPAs. Password manager overlays (1Password) can also interfere with the `type` action on name/password fields.

## `preview_eval` Has a 30s Hard Timeout

`mcp__Claude_Preview__preview_eval` times out at 30 seconds. Any asynchronous work inside the evaluated expression — especially `fetch()` calls to slow backend endpoints — will abort at 30s and return "preview_eval timed out" even if the request is still in flight server-side. Worst case: the server finishes the work (writes a row, sends an email, etc.) but the frontend never sees the response, and the eval reports failure.

### Common hit

Calling `POST /api/meal-plans/generate` or any other Claude Sonnet-backed endpoint from inside the preview. Full meal plan generation runs ~25 seconds end-to-end, which is just under the limit on a good day and over it on a slow day.

### Workaround

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

### Don't fight the timeout

If you see a timeout on `preview_eval`, don't retry the same eval hoping it's faster this time. Check the DB / server log first to see if the server-side work completed. If it did, you've just lost the response payload, not the effect. If it didn't, switch to the curl approach immediately.

### Critical: the request did NOT necessarily fail

A `preview_eval` timeout means the JavaScript Promise never resolved within 30s. It does NOT mean the underlying request failed. In the worst case, the server processed the request fully — wrote rows, spent money, triggered webhooks — and the eval just never saw the response. **Always check the DB for side effects before retrying a write-mutation request.** A write endpoint that timed out once and "looks like it failed" may have actually succeeded, and retrying it will create duplicates.

Real-world failure mode (Tedder Trainer Session 44, 2026-04-14): a `POST /api/meal-plans/generate` call from inside `preview_eval` timed out at 30s. I assumed the fetch hadn't landed and re-ran it via curl, which created a second plan. Then I deleted what I thought was the duplicate (the curl one, the most recent by id) — but the preview_eval request had also completed server-side, creating an earlier plan that I missed. That phantom plan sat in the DB and silently overrode the user's active meal plan in the app UI because the frontend's `/meal-plans/current` endpoint returns the newest matching row. The user was understandably alarmed.

**Recovery procedure** after a `preview_eval` write timeout:
1. Query the DB for any row the request would have created (by timestamp, user, or idempotency key).
2. If one exists, that IS the response you didn't see. Do not retry. Either use it or clean it up.
3. If none exists, only then retry via curl.
4. When cleaning up, sort by created_at / id and check ALL candidates, not just "the most recent one." Two phantom rows are possible if both preview_eval and your retry succeeded.

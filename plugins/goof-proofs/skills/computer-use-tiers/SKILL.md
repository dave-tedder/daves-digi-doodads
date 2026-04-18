---
name: computer-use-tiers
description: "Use when planning or executing browser/desktop automation involving the computer-use MCP, Claude-in-Chrome MCP, or Control Chrome MCP — picking the right tool, debugging Claude-in-Chrome login walls or iframe blind spots, working around Control Chrome's intermittent failures, or guiding Cowork agents that might fall back to a session-isolated tool."
---

# Computer-Use, Claude-in-Chrome, Control Chrome: Tool Selection and Gotchas

The computer-use MCP server injects its own guidance about tier system (read/click/full), the Chrome-MCP fallback for browsers, link safety, financial actions, and the request-access flow. This skill covers the operational lessons that aren't in that injection.

## Click-Tier Hybrid Workflow

When a desktop app is granted at "click" tier (visible left-clicks and scrolling, but typing and key presses blocked — applies to Terminal, IDEs, Shortcuts, etc.), plan for a hybrid workflow:

- Agent handles navigation clicks
- User types text input themselves
- Identify which steps require text input before starting and queue them up conversationally

Don't try to drive a click-tier app end-to-end with computer-use alone — typing will fail.

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
- **Control Chrome** for navigation (`open_url`, `execute_javascript`, `close_tab`). Operates on existing tabs with full cookie/session access.
- **Computer-use** for reading (`screenshot`). Gives visual understanding of the page (images, video thumbnails, attachment types, UI elements).

Control Chrome's `get_page_content` returns text only. It cannot distinguish images, videos, or attachment types. If the task requires visual understanding of page content, use computer-use screenshots instead of text extraction.

## Control Chrome Reliability

Control Chrome's `get_page_content` and `execute_javascript` intermittently fail with "Chrome is not running" even when `open_url` works and Chrome is visible. When this happens, use Claude-in-Chrome's `read_page`/`get_page_text` for content extraction, or `WebFetch` for public pages. WebFetch is particularly good for scouting multiple sites in parallel (can run 8-9 concurrent fetches).

## Claude-in-Chrome: Iframe Limitations

Claude-in-Chrome's `read_page` cannot see form fields inside iframes (e.g., embedded Wufoo, Typeform, Google Forms). The accessibility tree only returns elements from the parent page. Workaround: load the iframe source URL directly in a Claude-in-Chrome tab. For Wufoo forms, the direct URL pattern is `adminfoot.wufoo.com/forms/[form-id]/` (find the form ID in the iframe src attribute or page source).

## Cowork Agent Tool Fallbacks

Cowork agents will improvise tool fallbacks unless explicitly told not to. If a Cowork prompt uses Control Chrome and it fails, the agent may silently fall back to Claude-in-Chrome (which creates isolated tabs without login sessions). Any Cowork prompt should include hard rules about which tools NOT to fall back to when the primary tool fails.

## Claude-in-Chrome: form_input vs type on claude.ai

On claude.ai pages (and likely other complex SPAs), the `type` action frequently fails with "Detached while handling command." Use `form_input` with element ref IDs from `read_page` instead. It reliably sets values on text inputs and textareas. The `type` action is fine for simple pages but unreliable on React-heavy SPAs. Password manager overlays (1Password) can also interfere with the `type` action on name/password fields.

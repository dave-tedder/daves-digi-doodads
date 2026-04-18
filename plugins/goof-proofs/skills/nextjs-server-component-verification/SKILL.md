---
name: nextjs-server-component-verification
description: "Use when smoke-testing or verifying a Next.js App Router server component or Server Action from outside the browser — testing branching state machines without mutating production data, or curling a Server Action form endpoint and getting a 200 response that may not mean what it looks like."
---

# Verifying Next.js Server Components and Server Actions

Two verification techniques for Next.js App Router code that the regular curl-and-grep approach gets wrong.

## Verifying branching state machines via env-var short-circuits

When a server component has a branching state machine (e.g. `today` / `pending` / `empty` / `error`), reaching the rare branches often requires mutating production data. A cleaner technique: add one-line env-var short-circuits at the top of the state-loader function, restart `next dev` with each var set, curl + grep the rendered HTML to verify, then revert before committing.

```ts
async function loadState(): Promise<State> {
  try {
    const latest = await getLatest();
    if (!latest) return { kind: "empty" };
    if (isToday(latest)) {
      if (process.env.TEST_FORCE_PENDING) return { kind: "pending", fallback: latest };
      if (process.env.TEST_FORCE_EMPTY) return { kind: "empty" };
      if (process.env.TEST_FORCE_ERROR) return { kind: "error" };
      return { kind: "today", digest: latest };
    }
    return { kind: "pending", fallback: latest };
  } catch {
    return { kind: "error" };
  }
}
```

Verify per state with a single restart + curl:

```bash
pkill -f "next dev"; TEST_FORCE_PENDING=1 nohup npx next dev -p 3100 > /tmp/dev.log 2>&1 &
sleep 4; curl -s "http://localhost:3100/" -H "Cookie: ob-auth=$PW" -o /tmp/home.html
grep -o 'ascii-frame__top-tag">\[[^]]*\]' /tmp/home.html
```

Revert the env-var hooks before committing the rest of the component work. This avoids a test-only dependency shipping into production AND avoids destructive data mutations. Grep-assert on a distinctive class + state-tag string so each branch is confirmed, not eyeballed. Used successfully on Open Brain Session 49 (2026-04-17) to verify the DigestHeroCard's four states without deleting the day's digest row.

## Server Actions can't be smoke-tested with ordinary curl

Next.js App Router forms that use a Server Action render with `action={serverFn}` in JSX, which ships as HTML like:

```html
<form action="" encType="multipart/form-data" method="POST">
  <input type="hidden" name="$ACTION_ID_40c50e69..."/>
  ...
</form>
```

A standard `curl -X POST -d 'password=foo' ...` with `application/x-www-form-urlencoded` does NOT dispatch the server action. The server discards the malformed action request and re-renders the page, returning a 200 with the original form HTML. Both wrong and correct inputs come back as 200. No redirect, no cookie, no error — the action handler simply never ran.

Simulating a Server Action POST from curl requires:
- `Content-Type: multipart/form-data`
- A `Next-Action` request header (the action ID with prefix)
- The page-rendered `$ACTION_ID_...` hidden input value included in the form body

This is tedious enough that it almost never pays off. Two practical verification paths instead:

1. **Local dev + manual click.** Run `next dev`, hit the form in a real browser, inspect the response in devtools.
2. **Indirect production checks.** If the action writes a cookie, a row, or redirects, verify the side effect (cookie present on subsequent requests, row exists in the DB, `/login?error=1` renders correctly in-browser).

If someone reports "I curled my login endpoint and it returned 200 with the form HTML, so auth works," that's not actually verifying auth — they're watching Next.js ignore a malformed action dispatch. Observed on Open Brain dashboard 2026-04-17 when the same curl verification technique that worked for the login error state missed a live auth bypass entirely (the middleware wasn't running at all; the login form just happened to render identically to every other page because it used the same layout).

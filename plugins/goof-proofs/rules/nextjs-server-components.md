# Next.js Server Components: Event Handler Trap

Server components in Next.js (App Router) cannot pass event handlers as props to their children. This includes `<Link>` from `next/link`. Any `onClick`, `onChange`, `onMouseDown`, etc. attached to a child element triggers:

```
Error: Event handlers cannot be passed to Client Component props.
{href: ..., onClick: function onClick, ...}
                     ^^^^^^^^^^^^^^^^
```

This is fatal at request time — the page returns a 500 with an opaque digest. **Crucially, `npx next build` does NOT catch it.** The build passes clean and the error only surfaces when the route is actually rendered. The browser shows a generic "Application error" with a digest hash. The only place to see the real error is the runtime logs (Railway, Vercel, etc.).

This is easy to hit because `stopPropagation` is the obvious way to stop a click on a nested link from bubbling to a parent's click handler. It looks legal in JSX. It builds. It just dies on first request.

## How to fix

Two options, depending on what you actually need:

1. **Make the offending child its own client component.** If you actually need JS interactivity (state, refs, dynamic event handling), extract the interactive part into a `"use client"` component. The rest of the page can stay server-rendered.

2. **Use a CSS-only pattern instead.** For the common case of "make a whole card clickable while nested links still work," skip the JS entirely:

   ```tsx
   <div className="relative ...">
     {/* Background clickable overlay */}
     <Link href={cardHref} aria-label={title} className="absolute inset-0 z-10">
       <span className="sr-only">{title}</span>
     </Link>

     {/* Content layer: clicks fall through to overlay unless they hit a descendant <a> */}
     <div className="relative z-20 pointer-events-none [&_a]:pointer-events-auto">
       {header}
       {children}  {/* nested <Link>s navigate to their own targets */}
     </div>
   </div>
   ```

   How it works: the absolute background `<Link>` covers the card at `z-10`. The content layer at `z-20` has `pointer-events-none` so empty space lets clicks fall through to the overlay below. The arbitrary variant `[&_a]:pointer-events-auto` re-enables clicks on every descendant `<a>`, so nested links handle their own clicks normally. Overlay link and inner links are siblings (not nested anchors) — valid HTML.

   This pattern is fully server-component-compatible: no `"use client"`, no JS, no event handlers at all.

## Debugging tip

If a Next.js page returns a 500 with only a digest hash and `next build` passes, suspect this issue first. Check runtime logs (`railway logs --deployment` etc.) — the real error message and stack are there. Don't chase the digest.

## Verifying branching state machines in server components

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

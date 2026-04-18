---
name: nextjs-server-component-events
description: "Use when authoring a Next.js App Router server component that passes onClick, onChange, or any event handler to a child element (including <Link>), or when debugging a Next.js page that returns a 500 with an opaque digest hash but next build passes clean."
---

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

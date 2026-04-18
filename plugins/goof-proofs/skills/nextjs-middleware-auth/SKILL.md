---
name: nextjs-middleware-auth
description: "Use when implementing or debugging authentication in a Next.js App Router application — especially middleware-based password or token checks, when a protected route is serving content to anonymous traffic, when the redirect URL leaks an internal address, or when a deployed env var change exposes the site."
---

# Next.js Middleware and Auth Traps

Three distinct ways a Next.js middleware auth check can silently fail open. All three were live in production at the same time on one project (Open Brain dashboard, 2026-04-17). Combined exposure: every protected route served its content to anonymous traffic for weeks.

## 1. `middleware.ts` must live in `src/` if `src/app/` is the app directory

Next.js accepts `middleware.ts` at project root **only** when `app/` is at project root. If the app directory lives at `src/app/`, the middleware file must be `src/middleware.ts`. A root-level file is silently ignored — no warning, no build error, no "Middleware" line in the `next build` output. The app compiles and deploys, and every protected route serves its content to anonymous traffic.

**Detection:** run `npm run build` and look for a `ƒ Middleware  NN kB` line at the bottom of the route summary. If it's missing, middleware isn't being picked up.

**Fix:** `git mv middleware.ts src/middleware.ts`. Pure relocation, no logic change needed.

## 2. `cookie !== process.env.VAR` fails open when the env var is unset

```ts
// WRONG — fails open if DASHBOARD_PASSWORD is ever deleted
if (cookie !== process.env.DASHBOARD_PASSWORD) {
  return NextResponse.redirect(...);
}
```

If the env var is unset, both sides are `undefined`, and `undefined !== undefined` is `false`. The redirect never fires, every request falls through to the protected content. One deleted env var re-exposes the whole site with no loud failure.

```ts
// RIGHT — reject all three failure modes explicitly
const expected = process.env.DASHBOARD_PASSWORD;
const auth = request.cookies.get("ob-auth")?.value;
if (!expected || !auth || auth !== expected) {
  return NextResponse.redirect(...);
}
```

The same trap applies to any middleware that compares a cookie, header, or query param to an env var.

## 3. Route Handler `request.url` returns the internal bind on Railway

Next.js Route Handlers (`src/app/*/route.ts`) receive a standard `Request` object. On Railway's standalone Next.js deployment (`output: "standalone"` in `next.config.ts`), `request.url` dereferences the internal bind address (`http://localhost:8080`), not the public URL.

```ts
// WRONG on Railway — Location header leaks localhost:8080
return NextResponse.redirect(new URL("/login", request.url), 303);
```

Result in prod: `Location: https://localhost:8080/login`. The browser follows it literally and lands on a dead URL.

**Middleware does NOT have this problem.** `NextRequest.url` in middleware is built from the incoming request, preserving the client-facing host. So `new URL("/login", request.url)` works correctly in middleware and fails in route handlers for the same syntactic call.

**Fix options, in order of preference:**

1. Use a relative `Location` header. RFC 7231 §7.1.2 permits it; browsers resolve it against the request URL:

   ```ts
   const response = new NextResponse(null, {
     status: 303,
     headers: { Location: "/login" },
   });
   response.cookies.set("ob-auth", "", { maxAge: 0, path: "/" });
   return response;
   ```

2. Read the `x-forwarded-host` header and rebuild:

   ```ts
   const host = request.headers.get("x-forwarded-host") ?? request.headers.get("host");
   const proto = request.headers.get("x-forwarded-proto") ?? "https";
   return NextResponse.redirect(`${proto}://${host}/login`, 303);
   ```

Prefer option 1 unless the redirect target needs to change origin.

## Testing discipline

Unit tests and `next build` succeeding prove none of the three failure modes above. They are runtime-only failures that only surface when:
- Real HTTP traffic hits a deployed instance, and
- You curl with **no cookie** against a protected route.

Minimum verification after any auth change:

```bash
for p in / /thoughts /clients /login; do
  printf "%-12s " "$p"
  curl -sS -o /dev/null -w "status:%{http_code}  location:%{redirect_url}\n" \
    https://your-domain.example$p
done
```

Expect protected paths to return `307` with a `location` pointing at `/login`, and `/login` itself to return `200`. If everything is `200`, middleware isn't running.

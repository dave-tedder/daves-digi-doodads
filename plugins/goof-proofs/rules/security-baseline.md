# Security Baseline: Supabase & Railway Projects

These rules apply to any project using Supabase or Railway. Read this file at project start.

> **What's actually unique here:** most of this is general security hygiene that experienced developers already practice. The one item with a silent failure mode worth highlighting is the Supabase **"always destructure `{ error }` from writes"** rule — a single column type mismatch silently swallows the entire row operation with no thrown exception. That one's the gold; the rest is a sanity checklist.

---

## Supabase

- Enable Row-Level Security (RLS) on every new table. No exceptions.
- Never expose the `service_role` key client-side. Use `anon` key for browser/client code.
- Validate all inputs in Edge Functions before they hit the database.
- Never log PII (emails, phone numbers, payment info) in application logs or database audit columns.
- Use Supabase Auth for user identity. Don't roll custom session management.
- Storage buckets with user uploads must have RLS policies, not public access.
- Always check `{ error }` from `update()`, `insert()`, and `upsert()` calls. A single column type mismatch (e.g. sending a decimal to an integer column) silently fails the entire row operation. No exception thrown, no partial write. The error only surfaces if you destructure and check it.

## Railway

- All secrets go in environment variables, never hardcoded in source.
- Set up health check endpoints on every service so Railway can detect failures.
- Use private networking between Railway services when possible (no public exposure for internal APIs).
- Pin Docker base images to specific versions, not `latest`.

## General

- Never paste API keys or secrets into chat. Use `.env` files (gitignored).
- Error responses in production must be user-friendly. No stack traces, no internal paths.
- Rate limit any public-facing API endpoint.
- When in doubt about a destructive database operation, ask the user first.

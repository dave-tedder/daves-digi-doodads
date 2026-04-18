---
name: remote-asset-integrity
description: "Use when designing or debugging a pipeline that uploads files to remote storage (R2, S3, CDN) and subsequently triggers processing, rendering, or deployment from that storage — particularly for large files (>150MB), or when 'works locally but breaks in production' is suspected to come from a stale remote copy."
---

# Remote Asset Integrity

Any pipeline that uploads files to remote storage (R2, S3, CDN, etc.) then triggers processing from that storage must verify remote matches local before processing.

---

## The Rule

Never render, deploy, or process from remote storage without confirming the remote files match the local source of truth. A regenerated local file with a stale remote copy will produce silent, hard-to-diagnose failures.

## How to Apply

- **Before rendering/processing:** Compare local file sizes (or checksums) against remote. If they differ, abort with a clear error naming the stale files.
- **After regenerating any asset:** Re-upload it immediately, or flag it as dirty so the next pipeline step catches it.
- **In pipeline scripts:** Build the integrity check into the pipeline itself so it runs automatically. Don't rely on remembering to upload manually.

## Upload Method

- Files under ~150MB: presigned PUT URLs work fine.
- Files over ~150MB: use the S3 SDK `PutObjectCommand` directly. Presigned PUT via `fetch()` silently fails on large request bodies. This applies to R2, S3, and any S3-compatible storage.
- Presigned GET for downloads works at any file size.

## What This Prevents

- Audio/video sync failures from mismatched voiceover versions
- Rendering with outdated visual assets after regeneration
- Deploying stale code bundles after local changes
- Silent upload failures on large files (long-form video renders, 200MB+)
- Any scenario where "it works locally but breaks in production" is caused by forgotten uploads

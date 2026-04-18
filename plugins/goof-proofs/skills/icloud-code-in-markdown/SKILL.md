---
name: icloud-code-in-markdown
description: "Use when creating or editing markdown documentation stored on iCloud Drive that contains substantial fenced code blocks (more than a few lines), or when investigating missing/corrupted code blocks in synced markdown files."
---

# Don't Embed Code Blocks in Markdown on iCloud Drive

iCloud Drive sync (or editors syncing through iCloud) can strip or corrupt fenced code blocks in markdown files. This has been observed with large code blocks disappearing entirely from `.md` files after sync.

## Workaround

- Store code as standalone files (`.js`, `.ts`, `.py`, etc.) alongside the markdown documentation.
- Reference the file from the markdown doc instead of embedding the code inline.
- Example: "See SCRIPT-gmail-bridge.js in this directory for the full script."

## When This Applies

Any documentation stored on iCloud Drive that contains substantial code blocks (more than a few lines). Short inline code snippets are fine. The issue affects large fenced code blocks.

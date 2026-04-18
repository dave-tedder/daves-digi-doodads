---
name: notion-large-pages
description: "Use when `notion_retrieve_block_children` returns content exceeding token limits and gets saved to a temp file in `.claude/projects/`, or when working with Notion pages containing substantial content (long vendor lists, detailed notes, embedded draft emails) that might hit that limit."
---

# Notion Large Page Retrieval

When `notion_retrieve_block_children` returns content that exceeds token limits, the output gets saved to a temp file in `.claude/projects/`. Don't try to read the entire file sequentially.

## Workaround

1. Use Python or jq to search the dumped JSON for specific block IDs by keyword (e.g., search for blocks containing "liquor" or "metal").
2. Extract the block IDs you need to update.
3. Update individual blocks via `notion_update_block` with the specific block ID.

This is faster and more reliable than reading the full page content in chunks. Applies to any Notion page with substantial content (long vendor lists, detailed notes, embedded draft emails, etc.).

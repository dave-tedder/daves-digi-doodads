---
name: youtube-api-gotchas
description: "Use when writing or debugging code that uses the YouTube Data API — especially channel searches (which frequently return wrong channels by name match), processing video description text, or passing YouTube metadata downstream to an API that does strict JSON encoding (Anthropic API, etc.)."
---

# YouTube API Gotchas

## Channel Search Returns Wrong Channels

The YouTube Data API channel search frequently returns channels with similar names but completely different content. A search for "MommaStrong" returned the correct channel (15 public videos, content behind paywall) but also "Strong Mom" (543 videos, different creator). "EleniFit" returned a 2-video Spanish channel instead of the English fitness creator with 677 videos.

Always verify a channel ID before bulk operations:
1. Check total video count (`statistics.videoCount`)
2. Pull the first page of uploads and verify titles match expectations
3. Don't trust the first search result

## Video Descriptions Contain Broken Unicode

YouTube video descriptions can contain unpaired unicode surrogates (lone high or low surrogates without their pair). These pass through the YouTube Data API fine but crash JSON serialization in downstream APIs (confirmed with Anthropic's API: "no low surrogate in string").

Sanitize before sending to any API that does strict JSON encoding:
```
const safe = text.replace(/[\uD800-\uDFFF]/g, '');
```

The JS regex lookbehind approach (`(?<![\uD800-\uDBFF])[\uDC00-\uDFFF]`) does not work reliably in all Node versions. The simpler pattern above strips all surrogate code units, which removes emoji but is acceptable for metadata/prompt text.

Apply sanitization at the point of use (prompt construction), not just at fetch time, since data may be stored and retrieved later.

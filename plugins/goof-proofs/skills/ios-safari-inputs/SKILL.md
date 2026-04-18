---
name: ios-safari-inputs
description: "Use when building or debugging text input fields in a web app for mobile iOS Safari (chat inputs, forms, message composers), particularly if users report page zoom on focus, keyboard layout issues, the input being hidden behind the keyboard, or confusing form-assistant accessory bar UI."
---

# iOS Safari Input + Keyboard Gotchas

Three related traps that bite any mobile web app with a text input: focus-zoom, the form-assistant accessory bar, and the input-above-keyboard layout problem. None of them surface in desktop Safari or Chrome DevTools mobile emulation. All of them are fixable to varying degrees, but the accessory bar itself is OS-level and can't be fully removed from web code.

## 1. Focus zoom on small-font inputs

**Symptom:** Tapping an `<input>` or `<textarea>` causes the page to zoom in and stay zoomed.

**Cause:** iOS Safari auto-zooms on focus whenever the input's computed `font-size` is under **16px**. Tailwind's `text-sm` (0.875rem = 14px) and `text-xs` (0.75rem = 12px) both trigger it. `text-base` (1rem = 16px) does not.

**Fix:** Ensure the input renders at 16px or larger on mobile. Keep the rest of the layout smaller via responsive variants:

```tsx
<input
  className="text-base md:text-sm ..."  // 16px mobile, 14px desktop+
/>
```

Or set `font-size: 16px` inline if you need to escape Tailwind's scale.

The `user-scalable=no` viewport meta approach also works but degrades accessibility (prevents pinch-to-zoom on the entire page). Prefer the font-size fix.

## 2. The form-assistant accessory bar (↑ ↓ ✓)

**Symptom:** When an input is focused, iOS Safari shows a bar above the keyboard with up/down navigation arrows and a "Done" button. Users find it intrusive and often confused about whether they need to tap Done before submitting.

**Cause:** The accessory bar is a system-level UI rendered by iOS, not by the page. It's there for form-field navigation (skip between fields) and keyboard dismissal.

**What you can do:**
- `tabIndex={-1}` on sibling focusable elements (like a Send button next to the input) so iOS sees no other field to navigate to. This **removes or disables** the up/down arrows on most iOS versions.
- `enterKeyHint="send"` (or `"search"`, `"go"`, etc.) on the input changes the Return-key glyph from a generic checkmark/arrow to a platform-specific "Send" icon — users tap that to submit, no separate Send button required.
- `autoCorrect="off"`, `autoCapitalize="off"`, `spellCheck={false}`, `autoComplete="off"` on the input disable predictive-text suggestions, autocapitalize, and spell-check squiggles. Suggestions chips can also look like "confirmation" UI and cause the same confusion.

**What you can't do:**
- You cannot fully hide the accessory bar from web code. The "Done" button in particular persists in Safari regardless of markup. WebKit-specific hacks (`inputmode="none"` + custom keyboard) exist but are high-maintenance and degrade UX worse than the bar does.

Accept the residual "Done" button. The goal is to make it irrelevant to the workflow, not to remove it.

## 3. Pinning the input above the keyboard (Instagram-style)

**Symptom:** On mobile, when the keyboard opens, the text input either (a) gets hidden behind the keyboard, (b) stays at the bottom of the *original* viewport so there's awkward empty space above the keyboard, or (c) requires scrolling to reach.

**Cause:** `height: 100vh` is the **layout** viewport — fixed to the un-keyboarded screen height. When the keyboard slides up, the layout doesn't shrink, so a flex-column with `flex-1` messages + bottom-pinned input has the input land underneath the keyboard.

**Fix:** Use `dvh` (dynamic viewport height) instead:

```css
.chat-container { height: 100dvh; }
```

Or in Tailwind arbitrary-value syntax / inline style:

```tsx
<div style={{ height: "calc(100dvh - 120px)" }}>
```

`dvh` is *dynamic*: it excludes the on-screen keyboard from the viewport height calculation. When the keyboard opens, the container shrinks, and a bottom-pinned input (via `flex flex-col` with `flex-1` body + bottom-of-container form) floats right above the keyboard automatically.

Browser support: Safari 15.4+, Chrome 108+, Firefox 101+. All current iOS/Android versions support it. Fallback is `100vh` (old behavior). No JS needed.

## Detection / verification

These only reproduce on a real iOS device. Xcode Simulator doesn't always match real iPhone accessory-bar behavior. Chrome DevTools responsive mode doesn't simulate the iOS keyboard at all.

Verification checklist on a real iPhone:
1. Focus the input — page should NOT zoom.
2. Keyboard opens — input should sit directly above the keyboard, no empty space.
3. Return key on the keyboard shows "Send" (or whatever `enterKeyHint` specifies), and tapping it submits.
4. Accessory bar above the keyboard: the up/down arrows should be dimmed or absent. The "Done" button is expected to remain.

## Real-world use

Open Brain Session 53 (2026-04-17) shipped all three fixes in one commit (`d3f615b` on `dave-tedder/open-brain-dashboard`) for the neural-interface chat input on `brain.davetedder.com`. The 16px font bump, `100dvh` container, `enterKeyHint="send"`, and `tabIndex={-1}` on the SEND button together solved the reported zoom + checkmark + floating-input complaints. Verified working on iPhone.

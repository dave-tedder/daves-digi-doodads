---
name: css-hidden-attribute
description: "Use when authoring CSS rules that set `display:` on a class, or when debugging an element with the HTML `hidden` attribute set (or `el.hidden = true`) that still renders visible. Symptom: the page won't hide content, devtools shows the hidden attribute is present but ignored."
---

# HTML `hidden` Attribute vs. Class `display` Rules

The HTML `hidden` attribute relies on the user-agent stylesheet rule `[hidden] { display: none }`. Any class selector that sets `display: *` beats it on specificity (class > type/attribute in the UA sheet) and renders the element visible regardless of whether `hidden` is set.

## Symptom

An element with `hidden` set in markup or via `el.hidden = true` is still visible on the page. Toggling the attribute in devtools has no effect. The class applied to the element has a `display: flex`, `display: block`, `display: grid`, etc. rule.

## Fix

Add an explicit `[hidden]` override for any class that sets `display`:

```css
.my-overlay {
  display: flex;
  /* ... */
}
.my-overlay[hidden] {
  display: none !important;
}
```

The `!important` is belt-and-suspenders — the attribute-plus-class selector (0,2,0) already beats the class alone (0,1,0), but `!important` makes the intent obvious and survives future rule shuffling.

Alternative: don't use the `hidden` attribute at all. Toggle a `.hidden` class instead and keep visibility state in CSS.

## Real-World Failure

Tedder Trainer Session 40: the rest timer overlay had `.rest-timer-overlay { display: flex }` with no `[hidden]` override. The element was rendered with `hidden` in its HTML template and JavaScript only set `overlay.hidden = false` when a timer started. Result: the overlay was always visible the moment `renderActiveWorkout` injected the HTML, showing an empty "Rest 0s" modal with no handlers bound. The user was trapped. Shipped a wrong-cause fix first (thinking stale module state), then found the real cause by injecting the CSS in a preview and checking computed styles.

## Detection

When debugging a "this element won't hide" bug:
1. Check computed styles on the element. If `display` is anything but `none`, the `hidden` attribute is being overridden.
2. Look at which CSS rules apply in devtools. Any class rule with `display` is the culprit.
3. Test the fix in isolation: inject the class + `[hidden]` override in `preview_eval`, create two test elements (one with `hidden`, one without), and read `getComputedStyle(el).display` on both.

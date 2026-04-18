# iOS Print Gotchas

## `afterprint` fires on open, not close

iOS Safari fires the `afterprint` event when the print sheet **opens** (animates in), not when the user dismisses it. Any DOM cleanup in an `afterprint` handler (removing elements, stripping classes) destroys content before the print renderer captures it. The page flashes content then goes blank.

Never put cleanup logic in `afterprint` if the cleanup affects what the print renderer needs to see. Defer cleanup to a user interaction (modal close, next button click) or leave printable elements in the DOM if they're invisible on screen.

## `position: fixed` elements don't print on iOS

iOS print renderers don't reliably handle `position: fixed` or `display: flex` elements, even when `@media print` overrides them to `position: relative` / `display: block`. The renderer appears to use the computed layout from the screen context, not the print stylesheet override.

Don't try to print modals, overlays, or fixed-position containers in place.

## Working pattern: clone to a flat div

For printing content that lives inside a modal or fixed-position container:

1. Clone the content into a top-level `<div class="print-clone">` appended to `<body>`.
2. Style `.print-clone { display: none; }` on screen so it's invisible.
3. In `@media print`, hide everything (`body > * { display: none !important; }`) then show only `.print-clone` (`body > .print-clone { display: block !important; }`).
4. Clean up the clone on modal close or next print click. Never during print.

This sidesteps all timing races, positioning bugs, and `afterprint` event issues.

## Suppress decorative pseudo-elements in print

Pseudo-elements like `body::after` (noise textures, decorative overlays) with high `z-index` render in print and can cover content. Always add `body::after { display: none !important; }` to `@media print` blocks. Same applies to any `::before` / `::after` used for visual effects.

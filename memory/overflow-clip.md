---
title: "CSS overflow: clip"
tags:
- css
---

`overflow: clip` hides overflow like `hidden`, but unlike `hidden` it doesn't create a scroll container. This matters because you can apply it per-axis using `overflow-x` and `overflow-y` independently.

With `overflow: hidden`, browsers coerce the other axis to `auto` (scrollable). With `overflow: clip`, each axis stays exactly as you set it.

```css
/* Clip horizontally, allow vertical overflow to remain visible */
.container {
  overflow-x: clip;
  overflow-y: visible;
}
```

This combination (`clip` on one axis, `visible` on the other) is impossible with `hidden`.

## Why still use `hidden`?

- **Programmatic scrolling**: `hidden` creates a [scroll container](scroll-containers.md), so `element.scrollTo()` works. `clip` does not — it's not scrollable at all.
- **Browser support**: `clip` landed in Safari 16 (2022). If you need older browser support, `hidden` is the safer choice.

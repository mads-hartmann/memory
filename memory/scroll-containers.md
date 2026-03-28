---
title: Scroll containers
tags:
- css
---

A scroll container is any element whose content can be scrolled. An element becomes a scroll container when its computed `overflow` value is `auto`, `scroll`, or `hidden` (not `visible`, not `clip`).

See [CSS Overflow Module Level 3 §3](https://www.w3.org/TR/css-overflow-3/#scroll-containers) for the formal definition.

Once an element is a scroll container:

- It gets its own scrolling box and can be scrolled via JS (`scrollTo`, `scrollTop`, etc.)
- Scroll-related APIs work on it (`scrollHeight`, `scrollWidth`, `IntersectionObserver` root)
- `position: sticky` children stick relative to it
- It establishes a new block formatting context

`overflow: hidden` is a scroll container with the scrollbar suppressed — content is clipped visually but still scrollable programmatically. [`overflow: clip`](overflow-clip.md) is *not* a scroll container at all, which is why it can do per-axis control that `hidden` can't.

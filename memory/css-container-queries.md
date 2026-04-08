---
title: CSS container queries
tags:
- css
---

Container queries let you style elements based on the size of a parent container rather than the viewport. You mark a parent as a query container, then write `@container` rules that respond to that container's dimensions.

```css
.card-grid {
  container-type: inline-size;
  container-name: card-grid;
}

@container card-grid (min-width: 600px) {
  .card {
    grid-template-columns: 1fr 1fr;
  }
}
```

## `container-type: inline-size` changes layout

The main gotcha: setting `container-type: inline-size` (or `size`) on an element establishes a new **containment context** — a boundary that isolates the element's layout from its content's intrinsic sizes (see [CSS Containment Module Level 2](https://www.w3.org/TR/css-contain-2/#containment-context) for the formal definition). This changes how the element participates in layout.

Specifically, `container-type: inline-size` applies **inline-size containment** to the element. This means the element's inline size (width in horizontal writing modes) can no longer be determined by its content — it must come from context (parent constraints, explicit `width`, `flex`/`grid` sizing, etc.).

In practice this means:

- **Block-level elements** are usually fine. They already stretch to fill their parent's width, so containment doesn't change anything visible.
- **Inline-level and shrink-to-fit elements collapse to zero width.** Elements like `inline-block`, `float`, `fit-content`, or absolutely positioned elements normally size themselves based on their content. With inline-size containment, the browser ignores the content for sizing, and the element collapses unless you give it an explicit width.
- **Flex and grid items** can also be affected — see [caveats for flex layouts](#caveats-for-flex-layouts) below.

## Caveats for flex layouts

If a flex item's size comes from its content (e.g. `flex-basis: auto` with no explicit width), adding `container-type: inline-size` collapses it. The same applies to a flex container that sizes itself from its children — containment says "don't use content to determine size," but shrink-to-fit says "use content to determine size." They're mutually exclusive.

A common workaround is `flex: 1` so the element grows to fill its parent:

```css
.parent {
  display: flex;
}
.child {
  flex: 1;
  container-type: inline-size;
}
```

This works, but it changes the layout — the element no longer wraps its content, it fills available space. If the element is inside another flex or grid parent that constrains it, this is usually fine. If it's in a shrink-to-fit context all the way up, `flex: 1` resolves to zero and you're back to the same problem.

When neither an explicit width nor `flex: 1` fits the design, move the `container-type` to a different ancestor that already has a constrained width.

## `container-type: size` is stricter

`container-type: size` applies containment on both axes (inline and block). The element's height also can't depend on its content, so it collapses vertically too unless you set an explicit height. This is rarely what you want — `inline-size` is the common choice.

## Naming containers

`container-name` is optional but useful when you have nested containers. Without a name, `@container` queries match the nearest ancestor container. With a name, you can target a specific one.

```css
.sidebar { container-type: inline-size; container-name: sidebar; }
.main    { container-type: inline-size; container-name: main; }

@container sidebar (max-width: 300px) {
  .nav-link { font-size: 0.875rem; }
}
```

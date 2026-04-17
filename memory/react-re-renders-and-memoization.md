---
title: React re-renders and memoization
tags:
- react
- performance
---

## Re-render triggers

A component re-renders when:

1. Its own state changes (`useState`, `useReducer`).
2. A consumed context changes (`useContext`).
3. Its parent re-renders — **regardless of whether props changed**.

React does **not** diff props by default. A parent re-render cascades into all children unconditionally.

## Why it usually doesn't matter

Re-renders are cheap. React diffs the virtual DOM and only commits actual DOM mutations when something changed. The cost of a no-op re-render is just the component function executing + the diff. For most components this is negligible.

## When to memoize

- `React.memo()` — wraps a component to skip re-render when props are shallowly equal.
- `useMemo()` — memoizes an expensive computed value (or JSX) inside a component.
- `useCallback()` — memoizes a callback so it doesn't break a child's `React.memo`.

Only reach for these when profiling shows a bottleneck, or when a component is genuinely expensive (large lists, heavy computation).

## Children-as-props pattern

Passing children from above naturally avoids re-renders because the JSX was created by a parent that didn't re-render:

```tsx
function Layout({ children }) {
  const [count, setCount] = useState(0);
  return <div>{children}</div>; // children don't re-render
}
```

## React Compiler (React Forget)

The React Compiler auto-memoizes at build time:

- Component return values (like auto `React.memo`).
- Inline objects, arrays, and callbacks (like auto `useMemo`/`useCallback`).
- Hook dependencies.

With the compiler enabled, manual `useMemo`/`useCallback`/`React.memo` becomes unnecessary — the compiler inserts memoization where the data-flow analysis shows it's needed.

**Practical takeaway:** if adopting the React Compiler, skipping manual memoization is fine. Without it, you're relying on re-renders being cheap — which is true until it isn't.

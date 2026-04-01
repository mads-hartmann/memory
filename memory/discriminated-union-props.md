---
title: Discriminated union props
tags:
- typescript
- react
---

A discriminated union prop uses one property as a discriminant to narrow the types of sibling props. TypeScript calls this a "discriminated union" (also "tagged union"). It's useful for components where a mode flag changes the shape of the API.

```tsx
type SingleProps = {
  multiple?: false;
  value: string;
  onValueChange: (value: string) => void;
};

type MultipleProps = {
  multiple: true;
  value: string[];
  onValueChange: (value: string[]) => void;
};

type SelectProps = SingleProps | MultipleProps;

function Select(props: SelectProps) {
  // When multiple is true, TS knows value is string[]
  // When multiple is false/undefined, TS knows value is string
}
```

The discriminant (`multiple`) lets TypeScript infer the correct types for `value` and `onValueChange` at each call site.

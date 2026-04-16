---
title: Vitest --sequence.concurrent for stress-testing
tags:
- vitest
- testing
- flaky-tests
---

`--sequence.concurrent` runs all tests within a single file concurrently instead of sequentially. Useful for exposing shared-state bugs that cause flaky tests in CI.

## Usage

```bash
node_modules/.bin/vitest run path/to/file.test.ts --sequence.concurrent
```

## What it catches

- React Query cache leaking between tests
- ConfigCat / feature flag state shared across renders
- jsdom state pollution (e.g. one test's `screen` bleeding into another)
- Any test that implicitly depends on running after another test

## Example: gitpod-next dashboard

Running `use-environment-conversation.test.ts` with `--sequence.concurrent` reliably exposed a race where `waitFor(() => expect(isPending).toBe(false))` caught a transient state before the AI feature flag fully settled. The fix was to move the final assertion inside the same `waitFor` callback so both conditions must be true simultaneously.

## Caveat

Not all concurrent failures indicate real bugs — some test files legitimately share module-level mocks (e.g. `vi.mock`) that break under concurrency. Focus on failures that match patterns you've seen in CI flakes.

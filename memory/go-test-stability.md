---
title: Go test stability — avoid timing, scheduler, and global state
tags:
- go
- testing
- flaky-tests
- concurrency
---

If a test depends on timing, scheduler order, or process-global state, it will flake. Make async behaviour observable — via a channel, a callback, or a polled condition — and assert on that.

Credit: distilled from notes by Wouter Verlaek.

## 1. Don't `time.Sleep` to wait for async work — make the work observable

Sleeps assume the goroutine got scheduled and finished within N ms. CI under load makes that assumption wrong. Either have the production code expose a signal (a channel it closes when done), or poll a real condition with a generous deadline.

```go
// Bad — racy under CI load, the goroutine may not even have run yet
go doWork()
time.Sleep(100 * time.Millisecond)
if calls.Load() != 1 {
    t.Fatalf("expected 1 call, got %d", calls.Load())
}
```

```go
// Good (option A) — signal completion via a channel
done := make(chan struct{})
go func() {
    defer close(done)
    doWork()
}()

select {
case <-done:
case <-time.After(5 * time.Second): // safety net, not the assertion
    t.Fatal("doWork did not complete")
}
```

```go
// Good (option B) — poll a real condition with a generous deadline
deadline := time.Now().Add(2 * time.Second)
for calls.Load() == 0 && time.Now().Before(deadline) {
    time.Sleep(5 * time.Millisecond)
}
if got := calls.Load(); got == 0 {
    t.Fatalf("doWork was never called")
}
```

The deadline is a safety net so the test fails instead of hanging — it's not the thing you're asserting on.

## 2. Don't probe global/runtime state to infer what your code did — assert on signals it exposes

Things like `runtime.NumGoroutine()`, process-wide counters, or "did the heap grow" are polluted by everything else running in the test binary. They're fine as a quick local debug, not as a CI assertion. If you need to know that your component did something, have your component tell you.

```go
// Bad — process-global, parallel tests pollute the count
before := runtime.NumGoroutine()
h.startWorker()
h.Shutdown()
if runtime.NumGoroutine() > before {
    t.Fatal("worker leaked")
}
```

```go
// Good — each worker returns a done channel, closed on exit
done := h.startWorker() // <-chan struct{}
h.Shutdown()
select {
case <-done:
default:
    t.Fatal("worker did not exit before Shutdown returned")
}
```

Same idea applies to anything else you can't isolate from other tests — prefer a signal the component owns over a global metric you're reading from the side.

## 3. Don't use timed goroutines to trigger events mid-test — drive the ordering from the code under test

A common shape is "kick off the operation, then in a separate goroutine sleep N ms and cancel/inject an error". The scheduler decides which one wins, and CI load reorders things. Instead, trigger the event from a place where the ordering is guaranteed — e.g. from inside the mock/operation that the code under test is currently calling.

```go
// Bad — race: the cancel goroutine might fire before, during, or after
// the retry loop reaches its select
ctx, cancel := context.WithCancel(t.Context())
go func() {
    time.Sleep(5 * time.Millisecond)
    cancel()
}()
WithRetry(ctx, op, 5)
```

```go
// Good — the operation itself triggers the cancel, so the ordering is
// guaranteed: cancel happens-before the next ctx.Done() check
ctx, cancel := context.WithCancel(t.Context())
defer cancel()
op := func() (string, error) {
    cancel()
    return "", retryableErr
}
WithRetry(ctx, op, 5)
```

The general pattern: if you want "X happens after the system reaches state Y", hook into Y directly (a callback, the mock being invoked, a channel) rather than guessing at Y with a sleep.

## 4. Always include `t.Parallel()`

Top-level and inside every `t.Run`. Faster suite, surfaces shared-state bugs early.

```go
func TestThing(t *testing.T) {
    t.Parallel()
    for _, tc := range tests {
        t.Run(tc.Name, func(t *testing.T) {
            t.Parallel()
            // ...
        })
    }
}
```

## Rule of thumb

If a test depends on timing, scheduler order, or process-global state, it will flake. Make the async behaviour observable — via a channel, a callback, or a polled condition — and assert on that.

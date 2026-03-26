# Examples: Test Quality

Contrast pairs for the RED and GREEN steps of the TDD cycle.

---

## RED: Writing a Failing Test

```typescript
// Good: clear name, tests real behavior, one thing
test('retries failed operations 3 times', async () => {
  let attempts = 0;
  const operation = () => {
    attempts++;
    if (attempts < 3) throw new Error('fail');
    return 'success';
  };
  const result = await retryOperation(operation);
  expect(result).toBe('success');
  expect(attempts).toBe(3);
});
```

What makes it good: name describes behavior, uses real code, tests one thing.

```typescript
// Bad: vague name, tests mock not code
test('retry works', async () => {
  const mock = jest.fn()
    .mockRejectedValueOnce(new Error())
    .mockRejectedValueOnce(new Error())
    .mockResolvedValueOnce('success');
  await retryOperation(mock);
  expect(mock).toHaveBeenCalledTimes(3);
});
```

What makes it bad: name says nothing, asserts on mock call count instead of real behavior.

---

## GREEN: Minimal Implementation

```typescript
// Good: just enough to pass the test
async function retryOperation<T>(fn: () => Promise<T>): Promise<T> {
  for (let i = 0; i < 3; i++) {
    try { return await fn(); }
    catch (e) { if (i === 2) throw e; }
  }
  throw new Error('unreachable');
}
```

```typescript
// Bad: YAGNI — adding options no test requires
async function retryOperation<T>(
  fn: () => Promise<T>,
  options?: { maxRetries?: number; backoff?: 'linear' | 'exponential'; onRetry?: (n: number) => void }
): Promise<T> { ... }
```

What makes it bad: adds configurability the current test does not require. Features without tests are untested.

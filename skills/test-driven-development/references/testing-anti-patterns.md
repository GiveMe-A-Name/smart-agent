# Testing Anti-Patterns

**Load this reference when:** writing or changing tests, adding mocks or test utilities, or tempted to add test-only methods to production code.

---

## Core Principle

Every assertion must reveal something about the code under test. The subject of every test is the code — not the mocks, not the infrastructure, not the test utilities.

---

## Anti-Pattern 1: Asserting on Mock Behavior Instead of Real Behavior

```typescript
// ❌ BAD: verifying the mock exists, not the component
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});

// ✅ GOOD: test real behavior
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});
```

**Gate:** Before each assertion — ask: "If I deleted the code under test and kept only the mock setup, would this assertion still pass?" If yes: the assertion reveals nothing about the code. Delete it or remove the mock.

---

## Anti-Pattern 2: Test-Only Methods in Production Code

```typescript
// ❌ BAD: destroy() only called in tests
class Session {
  async destroy() {
    await this._workspaceManager?.destroyWorkspace(this.id);
  }
}

// ✅ GOOD: test utilities handle cleanup
// test-utils/session.ts
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) await workspaceManager.destroyWorkspace(workspace.id);
}
```

**Gate:** Before adding any method to a production class — ask: "Is this only called by tests?" If yes: move it to test utilities.

---

## Anti-Pattern 3: Mocking Without Understanding Dependencies

```typescript
// ❌ BAD: mock prevents the config write the test depends on
test('detects duplicate server', () => {
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
  }));
  await addServer(config);
  await addServer(config);  // Should throw — but won't, config was never written
});

// ✅ GOOD: mock at the right level
test('detects duplicate server', () => {
  vi.mock('MCPServerManager');  // Mock slow startup only; config write preserved
  await addServer(config);
  await addServer(config);  // Duplicate detected ✓
});
```

**The principle: mock at the boundary, not the method.** Boundaries are: external network calls, file system, clock, random, third-party services. Internals — your own domain logic and data transformations — should run real.

**Gate:** Before mocking any method — ask:
1. Is this a true boundary (external system, non-determinism, slow I/O)?
2. What side effects does the real method have that this test depends on?
3. If uncertain: run the test with the real implementation first. Observe what actually needs to happen. Then mock minimally at the right level.

Red flags: "I'll mock this to be safe" / "This might be slow, better mock it" / mocking without tracing the dependency chain.

---

## Anti-Pattern 4: Incomplete Mocks

```typescript
// ❌ BAD: missing fields downstream code uses
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' }
  // Missing: metadata that downstream code accesses
};

// ✅ GOOD: mirror the complete real API structure
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' },
  metadata: { requestId: 'req-789', timestamp: 1234567890 }
};
```

**Gate:** Before creating a mock response — check what fields the real API actually returns. Include all fields the system might consume downstream. If uncertain: include all documented fields.

---

## Anti-Pattern 5: Testing Implementation Details

```typescript
// ❌ BAD: test breaks when you rename a private method
test('processOrder calls validateInventory then chargePayment', () => {
  const spy = jest.spyOn(order, 'validateInventory');
  await processOrder(order);
  expect(spy).toHaveBeenCalledOnce();
});

// ✅ GOOD: test the observable behavior the caller sees
test('processOrder confirms payment when inventory is available', () => {
  const result = await processOrder(validOrder);
  expect(result.status).toBe('confirmed');
  expect(result.chargeId).toBeDefined();
});
```

**The sign**: your tests break when you rename an internal method, change an implementation strategy, or inline a helper — without changing any observable behavior. This means your tests are checking *how* the code works, not *what* it does.

Tests coupled to implementation details produce false failures on every refactor and give you a false sense of coverage. When you have to change tests while refactoring, the refactoring is unsafe.

**Gate:** After writing a test — ask: "Could this test remain identical if I completely rewrote the implementation, as long as the output behavior is the same?" If not, it is testing implementation details.

---

## When Mocks Are Too Complex

Warning signs:
- Mock setup is longer than the test logic
- You are mocking three or more levels of the call chain
- The test breaks when mock behavior changes but real behavior didn't change

These are symptoms of over-coupled production code, not limitations of testing. Consider: integration tests against real components are often simpler than elaborate mock setups for the same reason that the underlying code is too tangled.

---

## TDD Prevents These Anti-Patterns

Writing tests first forces you to see what the test actually needs before you add mocks. If you are testing mock behavior, you added mocks before watching the test fail against real code — a TDD violation. The discipline of watching the test fail first reveals what needs to be real and what can safely be replaced.

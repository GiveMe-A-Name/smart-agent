# Testing Anti-Patterns

**Load this reference when:** writing or changing tests, adding mocks or test utilities, or tempted to add test-only methods to production code.

## Core Principle

Mocks are isolation tools. The subject of every test is the code under test, not the mock.

## The Iron Laws

```
1. Every assertion must reveal something about the code under test
2. NEVER add test-only methods to production classes
3. NEVER mock without understanding the dependency
```

---

## Anti-Pattern 1: Assertions That Only Prove the Mock Exists

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

## Anti-Pattern 2: Test-Only Methods in Production

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

**Gate:** Before mocking any method — stop. Ask:
1. What side effects does the real method have?
2. Does this test depend on any of those side effects?
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

## When Mocks Are Too Complex

Warning signs:
- Mock setup is longer than the test logic
- Mocking everything just to make the test pass
- Test breaks when the mock changes

Consider: integration tests with real components are often simpler than elaborate mocks.

---

## TDD Prevents These Anti-Patterns

Writing tests first forces you to see what the test actually needs before adding mocks. If you are testing mock behavior, you added mocks without watching the test fail against real code first — a TDD violation.

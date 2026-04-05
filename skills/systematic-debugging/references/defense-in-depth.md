# Defense-in-Depth Validation

## Overview

When you fix a bug caused by invalid data, validating at one place feels sufficient — but that single check can be bypassed by different code paths, refactoring, or mocks.

**Core principle:** Validate at every layer data passes through. Make the bug structurally impossible.

Single validation: "We fixed the bug."
Multiple layers: "We made the bug impossible."

## The Four Layers

### Layer 1: Entry Point Validation
Reject obviously invalid input at the API boundary.

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
}
```

### Layer 2: Business Logic Validation
Ensure data makes sense for this specific operation.

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization');
  }
}
```

### Layer 3: Environment Guards
Prevent dangerous operations in specific contexts.

```typescript
async function gitInit(directory: string) {
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));
    if (!normalized.startsWith(tmpDir)) {
      throw new Error(`Refusing git init outside temp dir during tests: ${directory}`);
    }
  }
}
```

### Layer 4: Debug Instrumentation
Capture context for forensics when other layers fail.

```typescript
async function gitInit(directory: string) {
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack: new Error().stack,
  });
}
```

## Applying the Pattern

When you find a bug caused by bad data:

1. Trace the data flow — where does the bad value originate? Where is it used?
2. Map all checkpoints — list every place data passes through
3. Add validation at each layer — entry, business logic, environment, debug
4. Test each layer — try to bypass layer 1, verify layer 2 catches it

## Key Insight

Different layers catch different cases:
- Entry validation catches most bugs from callers
- Business logic catches edge cases from internal paths
- Environment guards prevent context-specific dangers (test pollution, wrong directory)
- Debug logging helps when all other layers fail

All four layers are needed. During testing, each layer will catch bugs the others miss — different code paths bypass entry validation, mocks bypass business logic, platform differences need environment guards.

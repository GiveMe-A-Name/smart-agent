# Root Cause Tracing

## Overview

Bugs often manifest deep in the call stack. Treat the visible failure line as a symptom until traced evidence shows it produced the invalid state. Trace backward through the call chain until the original trigger is identified, then fix at the source.

## When to Use

| Observable signal | Why this applies | First action |
|---|---|---|
| Error happens deep in execution, not at the entry point | The visible failure may be where invalid state is consumed, not where it is produced | Trace callers backward from the failing operation |
| Stack trace shows a long call chain | Framework, helper, or adapter frames can hide the project-owned behavior that created the state | Find the first project-owned behavior-changing caller |
| Invalid data appears at a boundary | The owning layer is the first observed location where the value was produced, transformed, or accepted without validation | Trace the value backward and record each producer or transformer |
| Failure occurs only for some test, caller, user, or input | A shared operation can be reached by many paths, and only one path may supply the bad state | Instrument the operation or bisect callers/tests |

## The Tracing Process

### 1. Observe the Symptom
```
Error: git init failed in /Users/user/project/packages/core
```

### 2. Find Immediate Cause
What code directly causes this?
```typescript
await execFileAsync('git', ['init'], { cwd: projectDir });
```

### 3. Ask: What Called This?
```typescript
WorktreeManager.createSessionWorktree(projectDir, sessionId)
  → called by Session.initializeWorkspace()
  → called by Session.create()
  → called by test at Project.create()
```

### 4. Keep Tracing Up
What value was passed?
- `projectDir = ''` (empty string)
- Empty string as `cwd` resolves to `process.cwd()`

### 5. Find Original Trigger
Where did the empty string come from?
```typescript
const context = setupCoreTest(); // Returns { tempDir: '' }
Project.create('name', context.tempDir); // Accessed before beforeEach runs!
```

## Adding Stack Traces

When you can't trace manually, add instrumentation:

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  console.error('DEBUG git init:', {
    directory,
    cwd: process.cwd(),
    nodeEnv: process.env.NODE_ENV,
    stack,
  });
  await execFileAsync('git', ['init'], { cwd: directory });
}
```

Use `console.error()` in tests — loggers may be suppressed.

Capture output:
```bash
npm test 2>&1 | grep 'DEBUG git init'
```

Look for: test file names, line numbers triggering the call, repeated patterns.

## Finding Which Test Causes Pollution

If the repository has no dedicated polluter-finding script, bisect the test set manually:

```bash
# Run half the candidate tests, then the other half.
# Keep halving until the polluter is isolated.
```

The key is the strategy, not a specific helper script: isolate the smallest test subset that still pollutes shared state, then trace why that test leaves residue behind.

## Key Principle

The immediate failing line is only the symptom until evidence shows it produced the invalid state. Before editing the consumer, trace backward until you can name the first observed location where the bad value, call, state, or assumption enters the system; make the primary fix there. Add a boundary check only when traced evidence shows another path can still supply the same invalid state; record that bypass path with the check [because fixing only the consumer can hide the crash while leaving the original trigger executable].

## Stack Trace Tips

- Use `console.error()` in tests — loggers may be suppressed
- Log before the dangerous operation, not after it fails
- Include: directory, cwd, environment variables
- Capture full call stack: `new Error().stack`

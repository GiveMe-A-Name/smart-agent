# Symptom vs Root Cause

The most common place to stop too early. A symptom names *what* is wrong. A root cause names *why* — the mechanism that produces the symptom.

---

## Example 1: Flaky Test

**Symptom observed:**
```
Expected: { status: 'pending' }
Received: { status: 'completed' }
```

### ❌ Stopped at symptom

> "The status is being set to 'completed' too early. I'll add a sleep to wait."

This names *what* is wrong, not *why*. The fix (sleep) doesn't address a mechanism — it just papers over timing. The test will keep being flaky.

### ✅ Found root cause

Investigation: Added logging before and after the event call. Observed that `EventEmitter.emit()` fires the handler synchronously — the handler runs *inline*, updating status before the `await` in the test resolves.

> "Root cause: Node's `EventEmitter.emit()` is synchronous. The handler runs before control returns to the test, so `status` is already `'completed'` by the time the assertion runs. Fix: use condition-based waiting to poll until the expected state, rather than assuming async sequencing."

The fix addresses the mechanism. The test stops being flaky.

---

## Example 2: Wrong Output

**Symptom observed:**
```
PaymentError: Connection timeout to payments.api.com
```

### ❌ Stopped at symptom

> "There's a connection timeout. I'll add retry logic."

This treats the symptom as the problem. Retries might mask the issue but won't prevent it.

### ✅ Found root cause

Investigation: checked recent changes (`git log --oneline -10`). Found a deploy 20 minutes ago that rotated SSL certificates. Checked the cert store used by the HTTP client — it's pinned to the old cert.

> "Root cause: the HTTP client is configured with a pinned certificate that expired in the last deploy rotation. The connection is refused at the TLS handshake, not due to network latency. Fix: remove cert pinning or update to the new cert."

Retries would never have fixed this.

---

## The Test

After investigation, ask: *can I predict what will happen if I make this specific change?*

- If yes: you have a root cause.
- If no: you have a hypothesis to test, not a root cause yet.

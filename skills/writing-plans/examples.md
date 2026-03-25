# Writing Plans — Examples

Each example shows: situation assessment → plan at the right depth.

---

## Tiny: Bug Fix

**Situation:** API returns 404 when an expired token is used, but should return 401.

**Assessment:**
- Size: Tiny — one condition check in one function, no interface change
- Nature: Bug fix — root cause identified: `auth_middleware.py:47` checks `token.exists()` before `token.is_valid()`, returns 404 on the `exists()` path
- Current state: `authenticate()` in `auth_middleware.py` handles all token errors; no other callers affected

**Plan:**

Root cause: `auth_middleware.py:47` returns 404 for missing tokens but the same 404 is also returned for expired tokens because expiry is checked after existence. Fix: check expiry on the token before returning 404.

Change `auth_middleware.py:47`:
```python
# Before
if not token:
    return Response(status=404)

# After
if not token:
    return Response(status=404)
if not token.is_valid():
    return Response(status=401)
```

Verify: `pytest tests/test_auth.py::test_expired_token_returns_401 -v` → PASS

---

## Small: Add a Feature

**Situation:** CLI `export` command needs a `--dry-run` flag that prints what would be exported without writing files.

**Assessment:**
- Size: Small — touches 3 files, stays in one domain (CLI layer), existing arg-parsing pattern applies
- Nature: Feature — `export.py` and its test already exist, just extending behavior
- Current state: `cli/export.py` uses `argparse`, writes via `FileWriter`; `FileWriter` has a `write()` method that can be conditionally skipped

**Plan:**

Goal: Add `--dry-run` to `export` command — prints output paths without writing.

Files:
- `cli/export.py` — add `--dry-run` arg, pass flag to writer
- `core/file_writer.py` — accept `dry_run=False` param in `write()`
- `tests/cli/test_export.py` — add test for `--dry-run` output

Steps:
- [ ] Add `parser.add_argument('--dry-run', action='store_true')` in `export.py:23`
- [ ] Pass `dry_run=args.dry_run` to `FileWriter` constructor
- [ ] In `file_writer.py:write()`: if `dry_run`, print path and return without writing
- [ ] Add test: mock `FileWriter.write`, assert not called when `--dry-run` passed

Verify: `python cli/export.py --dry-run output/` prints paths, no files created; `pytest tests/cli/test_export.py -v` PASS

---

## Medium: New Component

**Situation:** Add a webhook notification system — when a job completes, POST to a user-configured URL with the result.

**Assessment:**
- Size: Medium — introduces a new concept (webhooks), crosses job execution and config domains, needs new test design
- Nature: Feature — nothing like this exists; need to decide how webhook config is stored and how retries work
- Current state: `job_runner.py` fires `on_complete` event; config is stored in `config.yaml`; no HTTP client currently in use

**Plan:**

Goal: Fire a webhook POST when a job completes, with configurable URL and retry on failure.

Approach: Add `WebhookNotifier` as a subscriber to the existing `on_complete` event. Config lives in `config.yaml` under `webhooks:`. Use `httpx` for HTTP (already in dependencies).

File map:
- Create `notifications/webhook.py` — `WebhookNotifier` class, sends POST, retries up to 3x
- Create `tests/notifications/test_webhook.py` — unit tests with mocked HTTP
- Modify `config.yaml` schema — add `webhooks: [url, secret]` section
- Modify `job_runner.py:on_complete` — instantiate and call `WebhookNotifier` if configured

### Task 1: WebhookNotifier

Files:
- Create `notifications/webhook.py`
- Create `tests/notifications/test_webhook.py`

- [ ] Write failing test: POST is called with correct payload on job completion
- [ ] Implement `WebhookNotifier.notify(job_result)` — POST to URL, raise on non-2xx
- [ ] Write failing test: retries up to 3x on 5xx, gives up on 4xx
- [ ] Implement retry logic with exponential backoff
- [ ] Run: `pytest tests/notifications/test_webhook.py -v` → PASS

### Task 2: Config + Wiring

Files:
- Modify `config.yaml`
- Modify `job_runner.py`

- [ ] Add `webhooks:` section to config schema with `url` and `secret` fields
- [ ] In `job_runner.py:on_complete`: read config, instantiate `WebhookNotifier` if `webhooks.url` set
- [ ] Write integration test: job completion triggers webhook call
- [ ] Run: `pytest tests/ -v` → PASS

Verify: Run a test job with webhook URL pointed at `localhost` echo server, confirm POST received.

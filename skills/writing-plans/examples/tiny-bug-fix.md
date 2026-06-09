# Tiny: Bug Fix

**Situation:** An API returns 404 when an expired token is used. It should return 401.

**Assessment**
- **Size:** Tiny. One condition check in one function; no interface change.
- **Nature:** Bug fix. Root cause identified: `auth_middleware.py:47` checks `token.exists()` before `token.is_valid()`, so expired tokens follow the wrong branch.
- **Current state:** `authenticate()` in `auth_middleware.py` handles token errors; no other callers are affected.
- **Done state:** `pytest tests/test_auth.py::test_expired_token_returns_401 -v` passes.

## Human Review Section

> **Intent:** Fix the authentication bug where expired sessions show users a "not found" error instead of prompting them to log in again.
>
> **Summary (Layer 1)**
> - **Goal:** Expired tokens return Unauthorized instead of Not Found.
> - **Why:** Users should know their session expired, not think the endpoint is missing.
> - **End state:** Expired-token requests return 401.
> - **Impact scope:** Authentication behavior only.
> - **Size:** Tiny.

## Execution Plan

Root cause: `auth_middleware.py:47` checks token existence before validity. Add the validity check there so expired tokens return 401 before the missing-token path can return 404.

Verify: `pytest tests/test_auth.py::test_expired_token_returns_401 -v` -> PASS.

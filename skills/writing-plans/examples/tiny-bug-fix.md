# Tiny: Bug Fix

## Decision Brief

**Intent:** Fix expired sessions showing a misleading “not found” error so users know to log in again.

**Outcome:** Requests with expired tokens return 401 Unauthorized.

**Scope:** Expired-token authentication behavior only; valid and missing-token behavior stays unchanged.

**Assessment**
- **Size:** Tiny. One condition check in one function; no interface change.
- **Nature:** Bug fix. Root cause identified: `auth_middleware.py:47` checks `token.exists()` before `token.is_valid()`, so expired tokens follow the wrong branch.
- **Current state:** `authenticate()` in `auth_middleware.py` handles token errors; no other callers are affected.
- **Done state:** Expired tokens return 401, while valid and missing-token requests preserve their current behavior.

## Execution Plan

Root cause: `auth_middleware.py:47` checks token existence before validity. Add the validity check there so expired tokens return 401 before the missing-token path can return 404.

**Final verification:** `pytest tests/test_auth.py -v` -> PASS, including expired, valid, and missing-token cases.

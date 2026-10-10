# Introduction

A refresh-token endpoint exchanges a valid refresh credential for a new access token. Depending on policy, it may rotate the refresh token too.

# Why this concept exists

It lets a client continue a session after a short-lived access token expires without resubmitting the user's password.

# Real-world analogy

A renewal desk checks a longer-term voucher and issues a fresh temporary pass.

# Internal Working

The endpoint validates token signature/type/expiry and optional blacklist state. It then issues a new access token and, when rotation is configured, may return a replacement refresh token and invalidate the old one.

# Authentication Flow (where applicable)

```text
expired access -> client POSTs refresh -> refresh validation
-> new access / rotated refresh -> client replaces old credentials
```

# Syntax

```http
POST /api/token/refresh/
{"refresh":"<refresh-token>"}
```

# Complete Example 1

```python
# urls.py
from django.urls import path
from rest_framework_simplejwt.views import TokenRefreshView

urlpatterns = [
    path("api/token/refresh/", TokenRefreshView.as_view(), name="token_refresh"),
]
```

The built-in view handles refresh parsing and token issuance. Configure Simple JWT and authentication separately. HTTPS, abuse protection, and safe token storage are required; avoid exposing refresh tokens in logs.

# Complete Example 2

```python
from rest_framework.test import APIClient

client = APIClient()
pair = client.post("/api/token/", {"username": "alice", "password": "test-password"},
                   format="json").data
response = client.post("/api/token/refresh/", {"refresh": pair["refresh"]},
                       format="json")
assert response.status_code == 200
assert "access" in response.data
```

This integration test obtains a pair and refreshes it. Test rotation-enabled behavior separately: clients must replace the old refresh token if a new one is returned. Use test users and never print credentials.

# Complete Example 3

```python
SIMPLE_JWT = {
    "ROTATE_REFRESH_TOKENS": True,
    "BLACKLIST_AFTER_ROTATION": True,
}
```

Rotation provides a new refresh credential and blacklisting can invalidate the one presented, assuming blacklist support is installed and migrated. Review concurrency behavior: simultaneous refreshes can cause one request to invalidate credentials another request still expects.

# Advanced Example

Implement client-side single-flight refresh so concurrent failed requests share one refresh operation. On refresh failure, clear credentials and require reauthentication. Do not retry a refresh indefinitely.

# Under the Hood

Refresh is not a normal resource endpoint: it consumes a credential with broader lifetime. Rotation adds server-side revocation state and replay detection opportunities.

# Security Notes

Treat refresh tokens as high-value secrets. Store securely, rotate, limit lifetime, revoke on compromise, and apply rate limiting.

# Best Practices

- Send refresh credentials only to the refresh endpoint.
- Replace rotated tokens atomically on the client.
- Handle refresh failure by ending the local session.
- Test expiry, blacklist, replay, and concurrent requests.

# Common Mistakes

- Sending refresh tokens as ordinary API bearer credentials.
- Ignoring returned replacement refresh tokens.
- Retrying failed refresh calls forever.

# Debugging

Inspect request JSON key, token type/expiry, rotation settings, blacklist migrations, clock skew, and client token replacement.

# Interview Questions

1. What does refresh endpoint return? Usually a new access token; optionally a rotated refresh.
2. Why rotate refresh tokens? Reduce replay window and detect reuse.
3. What should client do after refresh failure? Clear credentials and reauthenticate.

# Practice Exercises

1. Add refresh route and successful integration test.
2. Test blacklisted/reused refresh failure.
3. Design a client strategy for simultaneous 401s.

# Chapter Summary

Refresh endpoints extend sessions without resending passwords but concentrate risk in a long-lived credential. Rotate, revoke, store, and retry it deliberately.

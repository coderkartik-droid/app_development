# Introduction

An access token is a short-lived credential used for API requests. A refresh token is a longer-lived credential used to obtain new access tokens without asking the user to enter credentials again.

# Why this concept exists

Short access lifetimes limit the usefulness of stolen tokens, while refresh tokens support a smooth user session.

# Real-world analogy

An access token is a temporary visitor pass; a refresh token is a more sensitive renewal voucher held securely and exchanged only at the badge office.

# Internal Working

The client sends the access token to resource endpoints. When it expires, the client sends the refresh token to a dedicated endpoint. The server validates it and issues another access token, optionally rotating/blacklisting refresh credentials.

# Authentication Flow (where applicable)

```text
login -> access + refresh
access -> protected resource -> permission -> response
expired access + refresh -> refresh endpoint -> new access (and possibly refresh)
```

# Syntax

```http
Authorization: Bearer <access-token>
```

# Complete Example 1

```python
from datetime import timedelta

SIMPLE_JWT = {
    "ACCESS_TOKEN_LIFETIME": timedelta(minutes=5),
    "REFRESH_TOKEN_LIFETIME": timedelta(days=1),
    "ROTATE_REFRESH_TOKENS": True,
    "BLACKLIST_AFTER_ROTATION": True,
}
```

Short-lived access reduces exposure; refresh rotation replaces a used refresh token, and blacklist-after-rotation invalidates the old one when blacklist support is installed and migrated. Confirm exact behavior against the pinned Simple JWT version and define cleanup for blacklist records.

# Complete Example 2

```python
from rest_framework_simplejwt.views import TokenRefreshView
from django.urls import path

urlpatterns = [
    path("api/token/refresh/", TokenRefreshView.as_view(), name="token_refresh"),
]
```

This exposes the refresh operation. It should accept only refresh credentials, be protected by HTTPS and abuse controls, and avoid logging request bodies. The endpoint does not make the refresh token safe to store in an exposed client environment.

# Complete Example 3

```python
response = APIClient().post(
    "/api/token/refresh/",
    {"refresh": refresh_token},
    format="json",
)
assert response.status_code == 200
assert "access" in response.data
```

The test assumes a valid token pair was created. Add cases for expired, malformed, revoked, and reused refresh tokens. Never print token values in test output.

# Advanced Example

Choose refresh-token storage based on client threat model. Browser apps often use secure HttpOnly cookies plus CSRF design; native clients need OS-backed secure storage. Define device-level revocation, theft response, expiry, rotation, and concurrent refresh behavior.

# Under the Hood

JWTs are signed claims, not encrypted secrets. Refresh validation and rotation may require server-side blacklist state; access verification can otherwise be stateless until expiry.

# Security Notes

Never use a refresh token as a bearer credential for ordinary resource APIs. Keep access tokens short-lived, protect refresh credentials more strongly, and plan revocation for account disablement and compromise.

# Best Practices

- Keep tokens separate by purpose and token type.
- Rotate refresh tokens and detect reuse where supported.
- Revoke sessions on logout/password reset as policy requires.
- Apply HTTPS, rate limits, and secure storage.

# Common Mistakes

- Giving access tokens multi-month lifetimes.
- Treating refresh tokens as harmless.
- Assuming deleting a client token revokes a copied token.

# Debugging

Check token type, expiry, rotation settings, blacklist app/migrations, clock sync, and response keys.

# Interview Questions

1. Why separate access and refresh tokens?
2. What does refresh rotation do?
3. Why can stateless access tokens be hard to revoke immediately?

# Practice Exercises

1. Configure short access and refresh rotation.
2. Test old refresh token rejection after rotation.
3. Write token theft/logout response procedures.

# Chapter Summary

Access and refresh tokens serve different risk windows. Secure refresh tokens as high-value credentials, rotate/revoke deliberately, and remember signed JWTs are not encrypted.

# Introduction

DRF's built-in TokenAuthentication uses a random token associated with a user. The client sends it as a bearer-like `Token` authorization credential on later requests.

# Why this concept exists

Token authentication allows clients that do not use Django browser sessions to authenticate API requests without sending the account password each time.

# Real-world analogy

A token is a replaceable access badge. It identifies the account, but anyone holding the badge may use its granted access until it is revoked.

# Internal Working

The client sends `Authorization: Token <key>`. DRF looks up the token, associates its user with the request, and then applies permission checks. The built-in token is generally long-lived and has limited rotation/expiry functionality.

# Authentication Flow (where applicable)

```text
Client token header -> TokenAuthentication -> token lookup -> request.user
-> permissions -> view -> serializer/database -> response
```

# Syntax

```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework.authentication.TokenAuthentication",
    ],
}
```

# Complete Example 1

```python
# settings.py
INSTALLED_APPS = [
    # existing apps...
    "rest_framework",
    "rest_framework.authtoken",
]

REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework.authentication.TokenAuthentication",
    ],
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
}
```

Add the authtoken app and run migrations to create token storage. Configure authentication and a permission requirement separately. This built-in token does not automatically expire; production requirements may call for a maintained token/JWT package or custom rotation/revocation policy.

# Complete Example 2

```python
# urls.py
from django.urls import path
from rest_framework.authtoken.views import obtain_auth_token

urlpatterns = [
    path("api-token-auth/", obtain_auth_token, name="api-token-auth"),
]
```

This endpoint accepts credentials and returns a token. Protect it with HTTPS, rate limiting, monitoring, and a suitable account lockout/brute-force strategy. The default response and token lifecycle are intentionally simple; do not assume refresh or expiry exists.

# Complete Example 3

```http
GET /api/profile/
Authorization: Token 0123456789abcdef0123456789abcdef01234567
```

The header scheme is `Token`, not `Bearer`, for DRF's built-in class. A valid token identifies the user; endpoint permissions still determine access. Never paste real tokens into shared documentation or logs.

# Advanced Example

For stronger lifecycle control, use a vetted solution that supports expiry, rotation, device/session records, and revocation. Store only hashed secrets where supported and define recovery/revocation procedures.

# Under the Hood

The default implementation stores a token row linked one-to-one with a user and retrieves it for each authenticated request. Token lookup can add a database query; caching or alternate designs require careful revocation semantics.

# Security Notes

Use HTTPS, keep tokens secret, revoke compromised tokens promptly, and avoid the default single permanent token for many devices if per-device revocation is required.

# Best Practices

- Require `IsAuthenticated` where data is private.
- Protect token-issuance endpoints against brute force.
- Define expiry/revocation and device management.
- Never log authorization headers.

# Common Mistakes

- Assuming built-in tokens expire automatically.
- Sending `Bearer` when configured for `TokenAuthentication`.
- Confusing valid authentication with authorization.

# Debugging

Check installed app/migrations, token row, header scheme, token value, authentication class, and permission result.

# Interview Questions

1. What header does DRF TokenAuthentication expect? `Authorization: Token ...`.
2. Does the built-in token expire by default? No.
3. What is its limitation for multiple devices? Default token model is typically one token per user.

# Practice Exercises

1. Add token auth to a protected profile endpoint.
2. Test invalid and revoked tokens.
3. Specify expiry and per-device revocation requirements.

# Chapter Summary

TokenAuthentication is simple for API clients, but its default token is long-lived and minimally managed. Pair it with HTTPS, access control, monitoring, and an explicit credential lifecycle.

# Introduction

Simple JWT is a commonly used Django REST Framework package that provides JWT authentication, token obtain/refresh views, and configurable token lifetimes and rotation behavior.

# Why this concept exists

Implementing cryptographic token issuance and validation by hand is risky. A maintained package supplies established primitives and integrates with DRF.

# Real-world analogy

Simple JWT is a tested badge office: it issues signed passes and checks them at API entrances, while your application still decides which rooms each pass holder may enter.

# Internal Working

The obtain endpoint authenticates credentials and issues tokens. `JWTAuthentication` validates a bearer access token on protected requests. Refresh endpoints validate refresh tokens and may rotate/blacklist them according to settings.

# Authentication Flow (where applicable)

```text
POST credentials -> TokenObtainPairView -> access + refresh
GET protected route + Bearer access -> JWTAuthentication
-> permissions -> view -> serializer/database -> response
```

# Syntax

```shell
python -m pip install djangorestframework-simplejwt
```

# Complete Example 1

```python
# settings.py
from datetime import timedelta

REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework_simplejwt.authentication.JWTAuthentication",
    ],
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
}

SIMPLE_JWT = {
    "ACCESS_TOKEN_LIFETIME": timedelta(minutes=5),
    "REFRESH_TOKEN_LIFETIME": timedelta(days=1),
}
```

Install the package first. The authentication class verifies access tokens; the global permission protects views; lifetimes bound credential exposure. Preserve other existing `REST_FRAMEWORK` settings and tune lifetimes to risk and usability requirements.

# Complete Example 2

```python
# urls.py
from django.urls import path
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    path("api/token/", TokenObtainPairView.as_view(), name="token_obtain_pair"),
    path("api/token/refresh/", TokenRefreshView.as_view(), name="token_refresh"),
]
```

The obtain view accepts credentials and returns a token pair; refresh accepts a refresh token and returns a new access token. Apply HTTPS, throttling, monitoring, and account protections to issuance routes. A logout endpoint requires a defined revocation strategy; token expiry alone is not immediate revocation.

# Complete Example 3

```python
from rest_framework.test import APIClient

client = APIClient()
token_response = client.post(
    "/api/token/",
    {"username": "alice", "password": "test-password"},
    format="json",
)
assert token_response.status_code == 200
access = token_response.data["access"]
client.credentials(HTTP_AUTHORIZATION=f"Bearer {access}")
```

This test obtains credentials and sets the client header for later requests. Use a test-created user, never a production credential. Add assertions for protected route access and invalid login. Exact token response keys follow package configuration.

# Advanced Example

Enable refresh rotation and blacklist support only after adding the blacklist app and migrations and deciding token storage/cleanup. Protect signing keys outside source control, and plan a key rotation procedure.

# Under the Hood

Simple JWT signs tokens using configured signing material and validates token type, signature, and time claims. Authentication typically resolves the user represented by the token, which may involve a database query.

# Security Notes

Use HTTPS, secure key management, short access lifetime, throttled login, refresh rotation/blacklisting when needed, and do not log tokens. Validate supported algorithms and claims for the deployment.

# Best Practices

- Pin and update the package version.
- Keep public token endpoints explicit.
- Test obtain, access, refresh, expiry, and revoked-token paths.
- Document client storage and logout limitations.

# Common Mistakes

- Forgetting to add the authentication class.
- Sending refresh token to protected APIs instead of access token.
- Enabling blacklist settings without app/migrations.

# Debugging

Check package installation, settings import, URL registration, bearer header, token type/expiry, user state, and blacklist migrations.

# Interview Questions

1. What does Simple JWT provide? JWT authentication and token obtain/refresh views.
2. Which token accesses protected APIs? Access token.
3. Does package installation alone protect endpoints? No; configure authentication and permissions.

# Practice Exercises

1. Install/configure Simple JWT in a sample project.
2. Test token obtain and protected endpoint access.
3. Add a documented expiry and rotation policy.

# Chapter Summary

Simple JWT integrates token issuance and verification into DRF. Correct configuration, lifecycle policy, secure key handling, and authorization remain application responsibilities.

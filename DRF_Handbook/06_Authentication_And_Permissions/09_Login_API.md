# Introduction

A login API verifies credentials and issues a session or token credential. With JWT, a token-obtain endpoint typically returns access and refresh tokens.

# Why this concept exists

Clients need a controlled way to establish identity before requesting protected resources.

# Real-world analogy

Login is the identity desk that checks a person's credentials and issues the appropriate temporary pass.

# Internal Working

The login view parses credentials, delegates password checking to Django's authentication system, applies throttling and account policy, then issues the configured credential. It should not reveal whether a username or password alone was wrong.

# Authentication Flow (where applicable)

```text
Client credentials -> login endpoint -> authenticate -> issue credential pair
-> client stores securely -> access token used on protected requests
```

# Syntax

```http
POST /api/token/
Content-Type: application/json
{"username":"alice","password":"..."}
```

# Complete Example 1

```python
# urls.py with Simple JWT
from django.urls import path
from rest_framework_simplejwt.views import TokenObtainPairView

urlpatterns = [
    path("api/token/", TokenObtainPairView.as_view(), name="token_obtain_pair"),
]
```

The package view authenticates credentials and issues access/refresh tokens. Add package configuration and HTTPS; protect login from brute force and monitoring abuse. Do not build password verification manually.

# Complete Example 2

```python
from rest_framework.test import APIClient

client = APIClient()
response = client.post(
    "/api/token/",
    {"username": "alice", "password": "test-password"},
    format="json",
)
assert response.status_code == 200
assert set(response.data) >= {"access", "refresh"}
```

The test sends JSON credentials and checks token keys. Create the user in test setup, never commit real passwords, and test invalid credentials and inactive users too.

# Complete Example 3

```python
REST_FRAMEWORK = {
    "DEFAULT_THROTTLE_CLASSES": [
        "rest_framework.throttling.AnonRateThrottle",
    ],
    "DEFAULT_THROTTLE_RATES": {"anon": "20/hour"},
}
```

Anonymous throttling can reduce basic credential-guessing volume, but it is not a complete anti-brute-force system. Configure trusted proxy behavior carefully and consider edge rate limits, monitoring, and account lockout policy without enabling denial-of-service against users.

# Advanced Example

Use generic error messages, audit login outcomes without storing passwords/tokens, alert on unusual patterns, and apply MFA or risk-based verification where required.

# Under the Hood

Django stores password hashes, not plaintext passwords. Authentication checks the submitted secret against the configured password hashers. The token issuer signs claims after identity validation.

# Security Notes

Use HTTPS, password hashers, rate limits, generic errors, MFA for sensitive contexts, and secure client-side credential storage. Never put passwords in URLs.

# Best Practices

- Delegate credential verification to Django/package.
- Throttle and monitor issuance endpoints.
- Keep response claims minimal.
- Define account disablement and token invalidation policy.

# Common Mistakes

- Returning different errors for unknown username versus wrong password.
- Logging credentials or token response.
- Forgetting login endpoints are public but abuse-sensitive.

# Debugging

Check URL, JSON parser, username field expectation, user password/status, token settings, throttle response, and system clock.

# Interview Questions

1. Why generic login errors? Reduce account enumeration.
2. Where should passwords be stored? As Django-managed salted hashes.
3. Is rate limiting sufficient alone? No, combine with monitoring and broader abuse controls.

# Practice Exercises

1. Test successful, invalid, and inactive-user login.
2. Add throttling and safe audit events.
3. Document client storage requirements.

# Chapter Summary

Login is a high-risk credential endpoint. Use established password verification and token issuance, protect it from abuse, and avoid revealing or logging secrets.

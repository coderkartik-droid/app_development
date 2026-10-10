# Introduction

Authentication answers “who is making this request?” In Django REST Framework (DRF), an authentication class examines request credentials and either associates a user (and sometimes an authenticator) with the request or leaves it anonymous. Authentication identifies; permissions decide what that identity may do.

# Why this concept exists

Private data and state-changing operations require a trustworthy identity. Without authentication, an API cannot distinguish one user's request from another's.

# Real-world analogy

A building receptionist checks an ID card and identifies a visitor. That does not automatically grant access to every room; a separate authorization rule still applies.

# Internal Working

DRF tries configured authentication classes in order. A successful class returns `(user, auth)`; DRF sets `request.user` and `request.auth`. If none authenticates, the request may be anonymous. Authentication does not itself reject a request unless policy requires a user.

# Authentication Flow (where applicable)

```text
Client -> credential (cookie/header) -> Authentication class
       -> request.user/request.auth -> Permission check -> View
       -> Serializer/Database -> Response
```

# Syntax

```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework.authentication.SessionAuthentication",
    ],
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
}
```

# Complete Example 1

```python
# settings.py
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework.authentication.SessionAuthentication",
    ],
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
}
```

**Line by line:** The first setting lists credential interpreters; session auth reads Django's session cookie. The second setting makes authentication a requirement by default. Existing public endpoints must explicitly opt into public access with `AllowAny`. Add secure HTTPS cookies and CSRF protection for browser sessions.

**Flow:** Client cookie -> SessionAuthentication -> `request.user` -> IsAuthenticated -> view -> serializer/database -> response. Removing the permission default can leave views accessible anonymously; adding authentication alone does not require login.

# Complete Example 2

```python
from rest_framework.permissions import IsAuthenticated
from rest_framework.views import APIView
from rest_framework.response import Response

class MyProfileView(APIView):
    permission_classes = [IsAuthenticated]

    def get(self, request):
        return Response({"username": request.user.get_username()})
```

`APIView` invokes authentication and permission checks before `get`. `request.user` is populated by the selected authenticator. The serializer is omitted because this tiny response is a primitive dictionary; production profile output should use an explicit serializer and avoid sensitive fields.

# Complete Example 3

```python
from rest_framework.test import APIClient

client = APIClient()
response = client.get("/api/me/")
assert response.status_code in (401, 403)
```

The exact unauthenticated status depends on the highest-priority authentication scheme and its challenge behavior. A robust test configures a specific scheme and asserts the documented result, then separately tests an authenticated identity.

# Advanced Example

Choose authentication based on client type and threat model: server-rendered browser sessions, service-to-service credentials, or short-lived signed tokens. Combine it with least-privilege permissions, rate limiting, secure transport, revocation/rotation strategy, and audit logging.

# Under the Hood

`APIView.initial()` performs content negotiation, version determination, authentication, permission checks, and throttling before calling the HTTP method handler. An authenticator must return a user/auth pair, return `None`, or raise an authentication exception.

# Security Notes

Never log credentials or bearer tokens. Require HTTPS, use safe cookie attributes, protect session-authenticated unsafe requests with CSRF, and treat identity claims as untrusted until cryptographically or server-side validated.

# Best Practices

- Distinguish identity from access policy.
- Set secure defaults globally and override public routes deliberately.
- Test anonymous, authenticated, and wrong-user cases.
- Use established authentication libraries and supported algorithms.

# Common Mistakes

- Thinking authentication automatically protects every view.
- Trusting a user ID submitted in JSON.
- Storing long-lived bearer tokens in logs or source code.

# Debugging

Inspect authentication class order, credential header/cookie, `request.user`, permission classes, CSRF behavior, and response challenge headers. Test through APIClient with realistic credentials.

# Interview Questions

1. What does authentication establish? The request's identity and credential context.
2. Does an authenticator decide object access? No, permissions do.
3. What are `request.user` and `request.auth`? Identity and authenticator-specific credential data.

# Practice Exercises

1. Protect a profile route with `IsAuthenticated`.
2. Test anonymous and authenticated requests.
3. Explain why CSRF matters for cookie-based authentication.

# Chapter Summary

Authentication establishes who sent a request. DRF performs it before permission checks, but a secure endpoint also needs authorization, transport protection, appropriate credential lifecycle, and tests.

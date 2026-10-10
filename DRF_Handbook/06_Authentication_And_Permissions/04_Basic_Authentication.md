# Introduction

HTTP Basic authentication sends a username and password in an `Authorization` header on each request, encoded with Base64. Base64 is not encryption.

# Why this concept exists

Basic auth is simple for controlled testing and some tightly managed service integrations, but repeated password transmission makes it unsuitable for most public production APIs.

# Real-world analogy

Basic auth is like writing a username and password on a note and placing that note in an envelope. The envelope must be protected in transit; encoding the note does not hide it.

# Internal Working

DRF decodes the header, authenticates credentials using Django's authentication system, and sets `request.user`. HTTPS is essential because anyone who intercepts the header can recover the credentials.

# Authentication Flow (where applicable)

```text
Authorization: Basic base64(user:password) -> HTTPS -> BasicAuthentication
-> request.user -> permission -> view -> serializer/database -> response
```

# Syntax

```http
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

# Complete Example 1

```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework.authentication.BasicAuthentication",
    ],
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
}
```

The authentication class parses Basic credentials; the permission requires a successfully authenticated user. Use HTTPS only. In many deployments, prefer session auth for browsers or a carefully selected token/OAuth solution for non-browser clients.

# Complete Example 2

```python
from rest_framework.authentication import BasicAuthentication
from rest_framework.permissions import IsAuthenticated
from rest_framework.views import APIView
from rest_framework.response import Response

class PrivateStatusView(APIView):
    authentication_classes = [BasicAuthentication]
    permission_classes = [IsAuthenticated]

    def get(self, request):
        return Response({"user": request.user.get_username(), "status": "ready"})
```

The view explicitly selects Basic auth and requires a user. `APIView` processes authentication before calling `get`. The example is for controlled use; do not send credentials over plain HTTP or store them in client code.

# Complete Example 3

```python
from rest_framework.test import APIClient

client = APIClient()
client.credentials(HTTP_AUTHORIZATION="Basic dGVzdDpwYXNz")
response = client.get("/api/private-status/")
assert response.status_code in (200, 401)
```

The encoded value is illustrative, not a valid credential guarantee. Use Django's test user and `client.force_authenticate()` for permission unit tests, or test actual Basic authentication with a real test password and HTTPS-aware integration environment.

# Advanced Example

If a legacy integration requires Basic auth, issue a dedicated low-privilege account, rotate its password, restrict network access, use HTTPS, monitor use, and avoid human account passwords.

# Under the Hood

The header is Base64-decoded into username/password and passed to Django authentication. Credentials are not stored as a reusable API token by DRF; the client resends them.

# Security Notes

Base64 provides no confidentiality or integrity. TLS protects in transit; logs, proxies, crash reports, and browser history must not capture authorization headers.

# Best Practices

- Use only over verified HTTPS.
- Prefer a dedicated service identity with least privilege.
- Rotate credentials and monitor access.
- Never embed secrets in public mobile/web bundles.

# Common Mistakes

- Calling Base64 “encryption.”
- Using Basic auth over HTTP.
- Reusing a human administrator password.

# Debugging

Check header scheme/encoding, credentials, Django user status, authentication class order, HTTPS/proxy setup, and `WWW-Authenticate` challenge.

# Interview Questions

1. Is Basic auth encrypted? No, only Base64 encoded.
2. Why HTTPS? It protects credentials in transit.
3. What is a safer use pattern? Dedicated restricted credentials with rotation and monitoring.

# Practice Exercises

1. Configure Basic auth for a local protected endpoint.
2. Test invalid credentials and missing credentials.
3. Write a deployment checklist that prevents plaintext credential transit.

# Chapter Summary

Basic auth is easy to understand but sends reusable credentials on every request. Restrict it to controlled HTTPS use and prefer mechanisms with an appropriate credential lifecycle for production clients.

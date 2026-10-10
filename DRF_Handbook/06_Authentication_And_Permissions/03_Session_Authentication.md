# Introduction

Session authentication reuses Django's server-side login session, identified by a browser cookie. It is particularly suited to browser clients hosted in the same trusted site environment.

# Why this concept exists

Traditional Django websites already manage users and sessions. DRF can use that identity for API requests without issuing a separate API token.

# Real-world analogy

After entering a secure office and receiving a temporary badge, a visitor presents that badge at each controlled door. The server checks the badge against its visitor register.

# Internal Working

The browser sends a session cookie; Django resolves it to a session and user. For unsafe methods, DRF's `SessionAuthentication` enforces CSRF validation for authenticated sessions.

# Authentication Flow (where applicable)

```text
Browser cookie -> Django session -> request.user -> CSRF check on unsafe method
               -> permissions -> view -> serializer/model/database -> response
```

# Syntax

```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework.authentication.SessionAuthentication",
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

SESSION_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
CSRF_COOKIE_SECURE = True
```

DRF uses Django session identity; the permission default requires login. Secure cookies require HTTPS and prevent JavaScript access to the session cookie. SameSite and domain policy should be deliberately configured for deployment. SessionAuthentication does not mean CSRF can be disabled.

# Complete Example 2

```python
from rest_framework.permissions import IsAuthenticated
from rest_framework.views import APIView
from rest_framework.response import Response

class CurrentUserView(APIView):
    permission_classes = [IsAuthenticated]

    def get(self, request):
        return Response({"username": request.user.get_username()})
```

The view runs only after session authentication and permission checks. The response exposes a minimal value, not the full user model. Add a URL route and login/session middleware in the Django project; otherwise `request.user` remains anonymous.

# Complete Example 3

```python
from rest_framework.test import APIClient

client = APIClient(enforce_csrf_checks=True)
client.login(username="alice", password="test-password")
response = client.post("/api/notes/", {"title": "Draft"}, format="json")
assert response.status_code == 403  # no CSRF token supplied
```

This test explicitly enables CSRF checks and demonstrates that a valid session does not bypass CSRF protection. A real positive test should fetch a page/token and send `HTTP_X_CSRFTOKEN`. Do not use real credentials in source.

# Advanced Example

For a same-site SPA, configure credentialed requests, CSRF token delivery, trusted origins, cookie domain, and SameSite behavior as a coherent deployment design. Cross-site cookies require careful browser policy and HTTPS.

# Under the Hood

SessionAuthentication obtains the Django user from the underlying request. If authenticated, it performs CSRF validation on unsafe HTTP methods; unauthenticated requests are left for later permission policy.

# Security Notes

Use HTTPS, secure/HttpOnly cookies, CSRF middleware, session expiry and rotation, and strong login protections. Do not place session cookies in local storage or logs.

# Best Practices

- Use sessions for same-origin browser apps when suitable.
- Keep CSRF middleware enabled.
- Configure cookie attributes per deployment.
- Test authenticated unsafe methods with CSRF enforcement.

# Common Mistakes

- Disabling CSRF because an endpoint is “an API.”
- Using session auth as a stateless mobile credential.
- Forgetting cross-origin credential and CSRF configuration.

# Debugging

Check session cookie presence, Django session middleware, login state, CSRF cookie/header, trusted origins, HTTPS, and authentication class configuration.

# Interview Questions

1. Where is session state stored? Server-side session storage, referenced by a cookie.
2. Why CSRF with session auth? Browsers automatically attach cookies.
3. Is session auth stateless? No.

# Practice Exercises

1. Protect a browser API with session authentication.
2. Write a CSRF-enforced POST test.
3. Document required cookie settings for production.

# Chapter Summary

Session authentication integrates DRF with Django browser sessions. Its security depends on CSRF defenses and careful cookie/HTTPS settings.

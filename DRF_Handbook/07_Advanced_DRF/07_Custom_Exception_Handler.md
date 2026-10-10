# Introduction

A custom exception handler centralizes transformation of DRF exceptions into a consistent response contract.

# Why this concept exists

Large APIs may need a shared error envelope, request ID, or stable machine-readable code across validation, permission, and not-found failures.

# Real-world analogy

A company-wide help desk uses one form for all incident reports while preserving the specific details needed to resolve each issue.

# Internal Working

DRF calls the configured handler with an exception and context. The handler should delegate to DRF's default handler first, then adjust recognized responses. Returning `None` means the exception is unhandled and will be re-raised.

# Authentication Flow (where applicable)

Credential/policy/view failure -> exception -> custom handler -> normalized response -> renderer -> client.

# Syntax

```python
REST_FRAMEWORK = {
    "EXCEPTION_HANDLER": "project.api.exceptions.api_exception_handler",
}
```

# Complete Example 1

```python
# project/api/exceptions.py
from rest_framework.views import exception_handler

def api_exception_handler(exc, context):
    response = exception_handler(exc, context)
    if response is None:
        return None
    response.data = {
        "errors": response.data,
        "status": response.status_code,
    }
    return response
```

The built-in handler preserves DRF exception semantics. The wrapper normalizes recognized responses without pretending unknown errors succeeded. Preserve headers and status; avoid losing `Retry-After` or authentication challenge headers. This envelope is a contract change and should be tested/documented.

# Complete Example 2

```python
# settings.py
REST_FRAMEWORK = {
    "EXCEPTION_HANDLER": "project.api.exceptions.api_exception_handler",
}
```

The dotted path must be importable. Merge this with existing DRF settings rather than replacing them. The handler applies to API view exceptions, not every exception in arbitrary Django code or middleware.

# Complete Example 3

```python
response = APIClient().post("/api/products/", {"price": "bad"}, format="json")
assert response.status_code == 400
assert "errors" in response.data
```

This verifies the global envelope for a validation error. Add cases for 401/403/404/429 and confirm headers remain intact. Ensure no secret or traceback appears in the normalized payload.

# Advanced Example

Add a request/correlation ID from trusted middleware context and map known domain exceptions to safe codes. Do not derive public error messages from arbitrary exception text.

# Under the Hood

The handler receives the exception and view/request context after `APIView.handle_exception` has applied authentication-challenge behavior. An unhandled exception can still become a server error through Django's normal error handling.

# Performance Notes

Keep the handler deterministic and cheap. Avoid database queries, network calls, or blocking logging sinks inside exception processing.

# Security Notes

Never return tracebacks, secrets, SQL, or raw internal exception strings. Ensure custom formatting preserves security-relevant headers and status.

# Best Practices

- Delegate to DRF's default handler first.
- Normalize only handled responses.
- Preserve headers and status codes.
- Test all expected error families and schema docs.

# Common Mistakes

- Returning HTTP 200 for every exception.
- Returning a fallback when default handler returns `None`.
- Dropping `WWW-Authenticate` or throttle headers.

# Debugging

Verify setting import path, handler invocation, returned status/headers, response body, and server-side traceback for unknown exceptions.

# Interview Questions

1. What does `None` from DRF's default handler mean?
2. Why delegate instead of rewriting exception mapping?
3. Which headers can custom handling accidentally lose?

# Practice Exercises

1. Add a stable error envelope and tests.
2. Preserve throttle and auth headers.
3. Add correlation IDs without exposing internal details.

# Chapter Summary

A custom exception handler is a central contract adapter. Delegate to DRF, preserve semantics and headers, and never convert unknown failures into success-shaped responses.

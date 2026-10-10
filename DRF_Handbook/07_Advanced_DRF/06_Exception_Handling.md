# Introduction

Exception handling converts expected failures—validation errors, missing objects, permission denials, and throttling—into HTTP responses.

# Why this concept exists

Clients need a stable way to understand failures, while server internals must not leak stack traces or database details.

# Real-world analogy

An exception handler is a service desk that translates an internal problem into a clear, safe message and escalation path.

# Internal Working

DRF catches supported exceptions raised during API view processing and calls its exception handler. Known API exceptions map to status codes and structured data; unhandled exceptions generally propagate as server errors.

# Authentication Flow (where applicable)

Client -> authentication/permission/view -> exception raised -> DRF handler -> safe status/body -> client.

# Syntax

```python
from rest_framework.exceptions import ValidationError, NotFound
```

# Complete Example 1

```python
from rest_framework.exceptions import ValidationError
from rest_framework.views import APIView
from rest_framework.response import Response

class DiscountView(APIView):
    def post(self, request):
        value = request.data.get("percent")
        if value is None or not 0 <= value <= 100:
            raise ValidationError({"percent": "Must be between 0 and 100."})
        return Response({"accepted": True, "percent": value})
```

`ValidationError` is an API exception handled as a 400 response. Production code should use a serializer for type coercion and field errors rather than comparing untrusted values directly. The view returns success only after validation.

# Complete Example 2

```python
from rest_framework.exceptions import NotFound

def get_order_or_404(order_id):
    try:
        return Order.objects.get(pk=order_id)
    except Order.DoesNotExist as exc:
        raise NotFound("Order not found.") from exc
```

The helper converts a model missing-row exception to a DRF 404. Prefer `get_object_or_404` or generic `get_object()` where suitable; scope the query before lookup to protect private resources.

# Complete Example 3

```python
response = APIClient().post("/api/discount/", {"percent": 140}, format="json")
assert response.status_code == 400
assert "percent" in response.data
```

This test asserts the public validation contract. A production test should also verify no state changed and cover wrong types and boundaries.

# Advanced Example

Define domain exceptions for expected business outcomes and map them deliberately to API exceptions at the boundary. Log unexpected exceptions with correlation IDs in server-side observability, not in response bodies.

# Under the Hood

`APIView.dispatch()` catches exceptions from initialization/handlers and calls `handle_exception`; configured exception handler transforms recognized exceptions into `Response`. Unhandled exceptions remain server errors.

# Performance Notes

Do not use exceptions for routine high-volume branching where a normal result is clearer. Keep error logging structured and avoid expensive synchronous reporting.

# Security Notes

Generic client-facing errors prevent leakage; detailed diagnostics belong in protected logs. Avoid echoing secrets or submitted passwords in validation messages.

# Best Practices

- Use DRF exception classes for expected API failures.
- Preserve field-level validation structure.
- Test status and body.
- Log unexpected failures once with safe context.

# Common Mistakes

- Returning success-shaped fallback after broad `except Exception`.
- Exposing raw SQL/database exception strings.
- Raising Python errors for normal validation without mapping.

# Debugging

Inspect exception type, handler configuration, traceback in protected logs, and serialized response. Reproduce with exact payload and identity.

# Interview Questions

1. How does DRF turn ValidationError into a response?
2. What should an unexpected exception response reveal?
3. Why prefer generic object lookup helpers?

# Practice Exercises

1. Add field validation and test its error body.
2. Map a domain conflict to a suitable API exception.
3. Verify an unexpected error is logged but not exposed.

# Chapter Summary

Handle expected failures with explicit DRF exceptions and stable client responses. Keep unexpected diagnostics server-side and avoid swallowing errors.

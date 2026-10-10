# Introduction

DRF error handling turns expected failures—invalid input, missing resources, denied access—into meaningful HTTP status codes and structured responses. Unexpected programming errors should remain visible to operators without exposing internals to clients.

# Why this concept exists

Clients need to distinguish invalid requests, missing records, authorization failures, and server faults so they can respond appropriately.

# Real-world analogy

A clear receipt says whether the request was incomplete, the item was unavailable, or the service encountered an internal problem.

# Internal Working

```text
known DRF exception -> exception handler -> status + structured response
unhandled exception -> server error path -> safe client response + operator logs
```

Serializer `ValidationError`, `NotFound`, and permission exceptions are handled by DRF's configured exception handler.

# Request Lifecycle

Exceptions raised in APIView dispatch/handler flow pass to `handle_exception`; the configured exception handler converts recognized exceptions. Rendering then formats the response. Errors outside the API view may be handled by Django middleware/server.

# Syntax

```python
serializer.is_valid(raise_exception=True)
```

# Complete Example 1

Use standard exceptions for missing data and validation:

```python
# api/views.py
from rest_framework import generics
from rest_framework.exceptions import NotFound
from .models import Book
from .serializers import BookSerializer


class BookDetailView(generics.RetrieveAPIView):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

    def get_object(self):
        try:
            return super().get_object()
        except Book.DoesNotExist as exc:
            raise NotFound("Book was not found.") from exc
```

```python
# api/urls.py
from django.urls import path
from .views import BookDetailView

urlpatterns = [path("books/<int:pk>/", BookDetailView.as_view(), name="book-detail")]
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("api.urls"))]
```

**Line-by-line:** The generic detail view provides lookup/serialization. `get_object` delegates to superclass and translates a known model missing exception to DRF `NotFound`; exception chaining preserves diagnostic cause. URL supplies primary key.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** route -> view policy -> queryset lookup -> missing-row exception -> DRF exception handler -> 404 response. For a found row, serializer outputs representation. **Internal working:** Generic `get_object()` normally raises `Http404` for no match, so this override is illustrative if custom lookup can raise `DoesNotExist`; avoid redundant handling in ordinary code. **Professional use case:** Translate domain-specific missing outcomes into consistent API errors. Broad `except Exception` would hide programming/database failures.

# Complete Example 2

Raise validation error:

```python
from rest_framework.exceptions import ValidationError


def perform_action(value):
    if value < 0:
        raise ValidationError({"value": ["Must be non-negative."]})
```

In real code, serializer validation is usually the proper input boundary. DRF maps the exception to a 400 response with structured field detail.

# Complete Example 3

Return a deliberate conflict when domain state prevents an operation:

```python
from rest_framework.exceptions import APIException


class AlreadyPublished(APIException):
    status_code = 409
    default_detail = "This article is already published."
    default_code = "already_published"
```

Raise it when the domain transition is invalid. Ensure clients can rely on documented status/code; do not leak internal state.

# Advanced Example

A custom exception handler can normalize error envelopes, but must preserve status and headers and should delegate to DRF's default handler for standard exceptions. Avoid returning stack traces or database text to clients.

# Under the Hood

APIView catches exceptions and calls `handle_exception`, which uses the configured exception handler. `APIException` carries status, detail, and code; validation exceptions preserve nested error structure.

# Performance Notes

Error handling has low cost. Logging high-volume expected validation failures at error severity can overwhelm observability; choose appropriate structured logging levels.

# Best Practices

- Use DRF exception classes and serializer errors.
- Return stable, actionable client details.
- Log unexpected failures with correlation IDs, not secrets.
- Preserve rollback when errors occur during transactions.

# Common Mistakes

- Catch-all exceptions that return 200 or generic success.
- Exposing tracebacks/SQL to clients.
- Converting every exception to 400, misclassifying server bugs.
- Returning inconsistent error envelopes across endpoints.

# Debugging

Inspect exception class, handler, status, response data, logs, and transaction outcome. Verify expected validation errors do not write records.

# Interview Questions

1. What handles DRF `ValidationError`? APIView exception handler.
2. Why not catch all exceptions? It hides defects and misreports errors.
3. What status for missing resource? Usually 404.
4. When use custom APIException? A stable domain error with appropriate status/code.

# Practice Exercises

1. Test invalid serializer input returns 400 and no write.
2. Test missing resource returns 404.
3. Implement a safe domain conflict exception and test code/status.

# Chapter Summary

Use framework exceptions for expected API failures and preserve unexpected failures for operator diagnosis. Errors should be accurate, structured, safe, and transactionally consistent.

# Introduction

Function-based API views (FBVs) use `@api_view` to adapt a Python function into a DRF endpoint. They are concise and make method handling visible.

# Why this concept exists

Small endpoints may not need a class hierarchy. A function communicates one focused request/response operation directly.

# Real-world analogy

An FBV is a one-person service desk with a clear checklist for a specific request.

# Internal Working

`@api_view` wraps the function in an APIView-compatible callable. It converts Django's request into a DRF `Request`, restricts methods, applies policies, and finalizes DRF responses.

# Request Lifecycle

```text
URL resolver -> @api_view wrapper -> method check/policies -> function
             -> Response -> renderer -> HTTP response
```

# Syntax

```python
@api_view(["GET", "POST"])
def items(request):
    ...
```

# Complete Example 1

The example validates an input label without database persistence.

```python
# api/views.py
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["POST"])
def create_label(request):
    name = request.data.get("name")
    if not isinstance(name, str) or not name.strip():
        return Response(
            {"name": ["A non-empty string is required."]},
            status=status.HTTP_400_BAD_REQUEST,
        )
    return Response(
        {"name": name.strip()},
        status=status.HTTP_201_CREATED,
    )
```

```python
# api/urls.py
from django.urls import path
from .views import create_label

urlpatterns = [path("labels/", create_label, name="create-label")]
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("api.urls"))]
```

**Line-by-line:** Imports provide HTTP constants, adapter, and response. The decorator allows POST. The function reads parsed body data, validates type and nonblank value, returns 400 for invalid input, otherwise normalized output and 201. URL modules mount it.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** Django URL resolver selects function -> DRF wrapper dispatches -> this compact example performs inline validation rather than a serializer -> no database -> response is rendered. **Internal execution:** request parsing occurs when accessing `request.data`; 405 is generated for unsupported methods. **Professional use case:** Tiny health/command endpoints; for growing schemas, extract validation into a serializer. Removing method list permits no intended method handling and breaks DRF adaptation.

# Complete Example 2

FBV can handle more than one method:

```python
@api_view(["GET", "DELETE"])
def resource(request, resource_id):
    if request.method == "GET":
        return Response({"id": resource_id})
    return Response(status=204)
```

This is complete only when imports and URL path converter are configured. The explicit method branch is easy to understand for a tiny endpoint, but grows unwieldy with many actions. A missing branch can lead to accidental behavior; allowed methods should match implementation.

# Complete Example 3

Attach a permission policy:

```python
from rest_framework.permissions import IsAuthenticated


@api_view(["GET"])
@permission_classes([IsAuthenticated])
def private_status(request):
    return Response({"user_id": request.user.pk})
```

Import `permission_classes` from `rest_framework.decorators`. Decorator order matters: `@api_view` should wrap the function and policy decorators should be applied in the documented order. Test anonymous access; never trust hidden client UI.

# Advanced Example

Use `@authentication_classes`, `@permission_classes`, and `@throttle_classes` only as deliberate endpoint-level overrides. For shared complex policies, class-based views or global defaults may be clearer.

# Under the Hood

The decorator creates an APIView subclass with handler methods that call the original function. This allows DRF dispatch/policy/exception/finalization behavior to apply.

# Performance Notes

FBV has no meaningful inherent performance advantage over equivalent APIView. Query count and serializer cost dominate.

# Best Practices

- Keep functions short and endpoint-specific.
- Use serializers for nontrivial validation.
- Return accurate status codes.
- Add tests for allowed and rejected methods.

# Common Mistakes

- Forgetting `@api_view`.
- Using `request.POST` for JSON.
- Catching all exceptions as success.
- Putting large CRUD workflows in one function.

# Debugging

Check decorator imports/order, URL resolver, allowed method, parser errors, permission policy, and response status.

# Interview Questions

1. What does `@api_view` do? Adapts function into DRF's API dispatch.
2. How restrict methods? Pass allowed methods to decorator.
3. When prefer FBV? Small, explicit API logic.
4. Does it automatically require login? Only if configured.

# Practice Exercises

1. Add GET-only health FBV and assert POST returns 405.
2. Validate a short input using serializer rather than inline checks.
3. Add an authenticated endpoint and anonymous test.

# Chapter Summary

FBVs are concise and explicit when endpoints are small. `@api_view` supplies DRF dispatch; use serializers and policies as complexity grows.

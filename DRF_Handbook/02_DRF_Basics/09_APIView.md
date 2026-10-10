# Introduction

`APIView` is DRF's class-based foundation for API endpoints. It provides method dispatch plus DRF request parsing, authentication, permissions, throttling, exception handling, and response finalization hooks.

# Why this concept exists

As endpoints need several HTTP methods and API-wide policies, a class-based boundary keeps each method handler clear and applies shared DRF behavior consistently.

# Real-world analogy

`APIView` is a staffed service counter with standard security checks and routing; `get` and `post` are separate service windows for different request actions.

# Internal Working

```text
APIView.as_view() -> Django route callable -> dispatch(method)
 -> initialize DRF Request -> policies -> get/post handler
 -> exception handling -> finalize Response -> renderer
```

# Request Lifecycle (where applicable)

Before the handler, DRF performs content negotiation, authentication, permissions, and throttling as configured. The matching method runs. A returned `Response` is finalized, while recognized exceptions become API error responses.

# Syntax

```python
class ItemList(APIView):
    def get(self, request):
        return Response([])
```

# Complete Example 1

This runnable in-memory collection view supports GET and POST. It is a teaching example, not persistent storage.

```python
# api/views.py
from rest_framework import status
from rest_framework.response import Response
from rest_framework.views import APIView

ITEMS = []
NEXT_ID = 1


class ItemList(APIView):
    def get(self, request):
        return Response({"results": ITEMS})

    def post(self, request):
        global NEXT_ID
        name = request.data.get("name")
        if not isinstance(name, str) or not name.strip():
            return Response(
                {"name": ["A non-empty string is required."]},
                status=status.HTTP_400_BAD_REQUEST,
            )
        item = {"id": NEXT_ID, "name": name.strip()}
        NEXT_ID += 1
        ITEMS.append(item)
        return Response(item, status=status.HTTP_201_CREATED)
```

```python
# api/urls.py
from django.urls import path
from .views import ItemList

urlpatterns = [path("items/", ItemList.as_view(), name="item-list")]
```

**Line-by-line:** Import DRF status, response, and `APIView`; the module list/ID simulate storage. `get` returns current items. `post` reads parsed body data, checks type and nonblank content, creates an ID/record, mutates the sample list, and returns 201. `as_view()` turns the class into Django's callable. **Execution flow:** request -> DRF initialize/policies -> `get` or `post` -> response -> renderer. **Internal working:** `APIView` wraps exceptions and applies configured policies. **Professional explanation:** Global mutable lists reset on process restart and are unsafe for multi-worker production; replace them with ORM/serializer logic. Removing `APIView` inheritance loses DRF dispatch; removing `as_view()` prevents proper URL registration.

# Complete Example 2

Explicitly reject unsupported verbs by not implementing their handlers:

```python
class ReadOnlyItems(APIView):
    def get(self, request):
        return Response({"results": []})
```

DRF dispatch returns 405 for a method without a handler, with an `Allow` header. Test this behavior; do not manually return 200 with an error message.

# Complete Example 3

Override view policy locally:

```python
from rest_framework.permissions import AllowAny


class PublicStatus(APIView):
    permission_classes = [AllowAny]

    def get(self, request):
        return Response({"status": "ok"})
```

This view permits unauthenticated access even if a restrictive global default exists. Make such exceptions deliberate and narrowly scoped.

# Advanced Example

For standard model CRUD, generic views or viewsets may reduce repetition. `APIView` is useful when request behavior is custom, but manually implementing every standard CRUD path can duplicate framework behavior.

# Under the Hood

`APIView.as_view()` returns a Django-compatible callable. DRF wraps the request, invokes policy hooks, dispatches by method, handles API exceptions, and finalizes renderer/context on the response.

# Performance Notes

APIView itself is not usually a bottleneck. ORM query count, serializer cost, authentication backend, and response size dominate; profile real workloads.

# Best Practices

- Keep each HTTP method handler focused.
- Use serializers for robust validation and representation.
- Configure permissions explicitly.
- Use database transactions for multi-step writes.

# Common Mistakes

- Forgetting `.as_view()`.
- Assuming `APIView` automatically requires authentication.
- Storing production data in process globals.
- Duplicating generic CRUD behavior unnecessarily.

# Debugging

Check method dispatch, permissions, parser errors, exception output, and URL registration. Use API tests for each method and anonymous/authenticated cases.

# Interview Questions

1. What does `APIView` add? DRF dispatch, request/response behavior, policy hooks, and exception handling.
2. Why use `as_view()`? It adapts a class into Django's callable view interface.
3. Does it automatically enforce authentication? Only according to configured defaults/view policy.
4. When choose generic views? Standard CRUD behavior where their abstractions fit.

# Practice Exercises

1. Add a detail `APIView` with GET and DELETE handlers.
2. Return 404 for unknown records and 204 after successful deletion.
3. Add an explicit permission class and test it.

# Chapter Summary

`APIView` provides a DRF-aware class dispatch pipeline. Use method handlers for custom behavior, serializers for data contracts, and explicit policies; choose generic views when they fit standard resource operations.

# Introduction

`APIView` is DRF's class-based view foundation. It maps HTTP methods to methods such as `get` and `post` while applying DRF request parsing, authentication, permissions, throttling, exception handling, and response finalization.

# Why this concept exists

Class attributes and method handlers make shared policy and multi-method behavior easier to organize than a growing function.

# Real-world analogy

APIView is a service counter with standard entry checks; each method is a different counter window for reading or changing a resource.

# Internal Working

```text
APIView.as_view() -> dispatch -> initialize_request -> initial policy checks
 -> get/post handler -> handle_exception -> finalize_response
```

# Request Lifecycle

URL routing calls the callable returned by `as_view()`. DRF initializes a `Request`, performs content negotiation and configured policy checks, dispatches by request method, then finalizes the returned response.

# Syntax

```python
class ItemList(APIView):
    def get(self, request):
        return Response([])
```

# Complete Example 1

This endpoint reads and creates in-memory notes to show dispatch. Replace process memory with a database for production.

```python
# api/views.py
from rest_framework import status
from rest_framework.response import Response
from rest_framework.views import APIView

NOTES = []
NEXT_ID = 1


class NoteList(APIView):
    def get(self, request):
        return Response({"results": NOTES})

    def post(self, request):
        global NEXT_ID
        title = request.data.get("title")
        if not isinstance(title, str) or not title.strip():
            return Response({"title": ["Required."]}, status=400)
        note = {"id": NEXT_ID, "title": title.strip()}
        NEXT_ID += 1
        NOTES.append(note)
        return Response(note, status=status.HTTP_201_CREATED)
```

```python
# api/urls.py
from django.urls import path
from .views import NoteList

urlpatterns = [path("notes/", NoteList.as_view(), name="note-list")]
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("api.urls"))]
```

**Line-by-line:** Imports bring DRF status, `Response`, and `APIView`. `NOTES`/`NEXT_ID` are teaching-only process state. `get` returns a collection; `post` parses data, validates a title, assigns ID, appends and returns 201. `as_view()` exposes class dispatch to URL routing.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** route -> APIView dispatch -> handlers -> inline validation (no serializer) -> in-memory list (no database) -> Response renderer. **Internal execution:** DRF invokes `get` or `post` based on method; missing methods receive 405. **Professional use case:** Custom API behavior with shared class-level policies. Removing in-memory globals loses persistence and is unsafe across workers; replace them with ORM operations and serializers.

# Complete Example 2

Set policy on a view:

```python
from rest_framework.permissions import IsAuthenticated


class PrivateDashboard(APIView):
    permission_classes = [IsAuthenticated]

    def get(self, request):
        return Response({"user": request.user.pk})
```

Authentication identifies a caller; `IsAuthenticated` gates the handler. Configure authentication classes/global settings as required. Without the permission attribute, effective global defaults apply, which may be more permissive.

# Complete Example 3

Use `get` and `delete` methods for a detail endpoint:

```python
class ItemDetail(APIView):
    def get(self, request, item_id):
        ...

    def delete(self, request, item_id):
        ...
```

Register a typed URL with `<int:item_id>`. Each method should perform an authorized lookup and return 404 for a missing record. Avoid duplicating lookup logic; generic views may fit better.

# Advanced Example

Override `initial`, `handle_exception`, or `finalize_response` only when a cross-cutting API requirement cannot be expressed through standard policies/settings. Such overrides affect many behaviors; preserve superclass contracts and test all branches.

# Under the Hood

`APIView.as_view()` wraps Django's view callable and marks API behavior. `dispatch` resolves handler methods, applies request initialization and policies, invokes handler, and converts known exceptions before final response rendering.

# Performance Notes

APIView overhead is typically small. Keep database access efficient, use pagination, and avoid repeated relation lookups.

# Best Practices

- Use APIView for custom multi-method logic.
- Keep validation in serializers.
- Scope data access by request identity.
- Prefer generic views for standard CRUD.

# Common Mistakes

- Forgetting `.as_view()`.
- Assuming APIView enforces authentication by itself.
- Overriding dispatch internals without preserving behavior.
- Using process-global mutable data in production.

# Debugging

Inspect route, method handler, effective permissions, parser, exception handling, and response finalization. Test each HTTP method and policy.

# Interview Questions

1. What is `APIView`? Class-based DRF request handler with API lifecycle integration.
2. What does `as_view()` provide? Django-compatible callable.
3. What does `dispatch` do? Selects handler based on HTTP method after initialization/policy checks.
4. When use APIView over generic? When behavior is custom and doesn't fit standard mixins cleanly.

# Practice Exercises

1. Build GET/POST handlers with serializer validation.
2. Add authenticated policy and test anonymous access.
3. Implement a detail DELETE with 404 and 204 cases.

# Chapter Summary

APIView offers explicit class-based method dispatch and DRF policy hooks. Use it for custom workflows, while serializers own data validation and generic views handle conventional operations.

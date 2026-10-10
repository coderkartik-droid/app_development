# Introduction

View and ViewSet mistakes often come from assuming that routing, generic behavior, or a hidden client control provides security. DRF views execute server-side and must enforce the full endpoint contract themselves.

# Why this concept exists

A route can be valid while exposing too many actions, too many records, or unsafe writes. Understanding framework defaults and hooks prevents security and correctness failures.

# Real-world analogy

A locked front door does not secure every room if internal doors are left open; each route and resource operation needs the right access rule.

# Internal Working

```text
router exposes action -> policy checks -> queryset scope -> serializer -> persistence
```

Security and correctness are layered; a ViewSet/router does not automatically infer domain permissions.

# Request Lifecycle

A request may fail before view logic due to negotiation/authentication, or after dispatch due to parsing/validation/ORM. Debug each stage in order.

# Syntax

```python
def get_queryset(self):
    return Document.objects.filter(owner=self.request.user)
```

# Complete Example 1

Correct owner-scoped list and create:

```python
# documents/views.py
from rest_framework import generics
from rest_framework.permissions import IsAuthenticated
from .models import Document
from .serializers import DocumentSerializer


class DocumentListCreateView(generics.ListCreateAPIView):
    serializer_class = DocumentSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Document.objects.filter(owner=self.request.user)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

```python
# documents/urls.py
from django.urls import path
from .views import DocumentListCreateView

urlpatterns = [path("documents/", DocumentListCreateView.as_view())]
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("documents.urls"))]
```

**Line-by-line:** The generic view supplies list/create. Serializer defines fields. Authentication is required. Queryset filters visible objects; perform_create sets trusted owner. Route exposes endpoint.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** request -> auth -> scoped queryset or serializer validation -> ORM -> response. **Internal execution:** Detail access using this queryset will return 404 for other users' records. **Professional use case:** Private documents. Removing either auth or scoping can create exposure; both are needed because authentication alone does not establish ownership.

# Complete Example 2

Avoid returning raw models:

```python
return Response(DocumentSerializer(document).data)
```

Serializers define representation. Returning a model instance directly is not renderable API output and may expose fields if converted carelessly.

# Complete Example 3

Do not enable all ModelViewSet actions when only reading:

```python
class PublicDocuments(viewsets.ReadOnlyModelViewSet):
    ...
```

This exposes only list/retrieve actions, unlike ModelViewSet's full CRUD. Still filter records and define permissions.

# Advanced Example

Avoid overriding `dispatch`/`create` for simple customization. Use `get_queryset`, `get_serializer_class`, `perform_create`, permission hooks, or action decorators to retain framework behavior.

# Under the Hood

Routers only create URL/action mappings. Permissions run as configured; object permissions may need `get_object`; queryset filtering controls list visibility. These mechanisms are related but not interchangeable.

# Performance Notes

Unbounded endpoints and N+1 queries are common. Paginate and align queryset optimization with serializer access.

# Best Practices

- Test anonymous, owner, non-owner, and staff behaviors.
- Scope list and detail queries.
- Restrict action set intentionally.
- Preserve framework mixin behavior.

# Common Mistakes

- Assuming authentication means object ownership.
- Leaving `.all()` queryset for private resources.
- Exposing create/update/delete by default.
- Returning success after swallowed database errors.

# Debugging

Inspect actual registered routes, action map, permission classes, queryset, serializer fields, and logs. Use two distinct test users to expose isolation bugs.

# Interview Questions

1. Does IsAuthenticated ensure record ownership? No.
2. What controls list visibility? Queryset filtering.
3. What does router registration do? URL/action mapping, not permissions.
4. How restrict methods? Use read-only viewsets or selected mixins.

# Practice Exercises

1. Fix an unrestricted private ViewSet.
2. Add tests for cross-user list and detail.
3. Compare read-only and full ModelViewSet routes.

# Chapter Summary

Views and routers do not provide automatic domain security. Scope querysets, choose actions intentionally, serialize output, and test access boundaries with multiple users.

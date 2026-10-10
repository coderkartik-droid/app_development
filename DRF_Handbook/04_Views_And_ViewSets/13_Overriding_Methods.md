# Introduction

DRF generic views and ViewSets expose well-defined hooks and methods for customization. Overriding the narrowest suitable hook preserves framework behavior better than replacing a whole handler.

# Why this concept exists

Applications need request-aware querysets, server-assigned values, action-specific serializers, or custom transitions. Hooks let these changes fit into the framework's normal lifecycle.

# Real-world analogy

A standard workflow has approved adjustment points: add the user's identity at intake rather than rebuilding the entire processing line.

# Internal Working

```text
generic handler -> get_queryset/get_serializer_class/perform_create hook
                -> standard mixin flow continues
```

Overriding `list`, `create`, or `dispatch` can bypass pagination, headers, exception handling, or save behavior if superclass logic is omitted.

# Request Lifecycle

The hook is invoked at a defined stage: queryset before lookup/listing, serializer selection before validation/representation, `perform_create` after validation at save time.

# Syntax

```python
def perform_create(self, serializer):
    serializer.save(owner=self.request.user)
```

# Complete Example 1

Use narrow hooks to scope reads and assign owner. Assume a `Note` model and serializer.

```python
# notes/views.py
from rest_framework import generics
from rest_framework.permissions import IsAuthenticated
from .models import Note
from .serializers import NoteSerializer


class NoteListCreateView(generics.ListCreateAPIView):
    serializer_class = NoteSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Note.objects.filter(owner=self.request.user).order_by("-created_at")

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

```python
# notes/urls.py
from django.urls import path
from .views import NoteListCreateView

urlpatterns = [path("notes/", NoteListCreateView.as_view(), name="note-list")]
```

```python
# project/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("notes.urls"))]
```

**Line-by-line:** The generic view supplies GET/POST. Authentication is required. `get_queryset` scopes list and detail behavior to current user. `perform_create` injects owner after serializer validation. URL exposes collection.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** request -> generic handler -> `get_queryset` for GET or serializer validation for POST -> `perform_create` -> ORM -> serializer -> Response. **Internal working:** Hooks preserve pagination, response headers, and standard errors. **Professional use case:** User-owned records. Removing `get_queryset` can permit cross-user access; overriding `create` just to set owner risks bypassing standard behavior.

# Complete Example 2

Select serializer by action:

```python
def get_serializer_class(self):
    if self.request.method == "POST":
        return NoteWriteSerializer
    return NoteReadSerializer
```

In a ViewSet, checking `self.action` is often clearer. Ensure each serializer is valid for its operation and test resulting output. A missing fallback causes runtime errors for unexpected actions.

# Complete Example 3

If custom response behavior requires overriding `create`, preserve the base flow when possible:

```python
def create(self, request, *args, **kwargs):
    response = super().create(request, *args, **kwargs)
    response.data["notice"] = "Saved"
    return response
```

This keeps validation, save hooks, success headers, and status behavior. Do not mutate response data this way if it may be immutable or cached in your version; a safer approach can construct a new `Response` from a copied mapping. Test API contract explicitly.

# Advanced Example

Use `get_permissions`, `get_throttles`, and `get_serializer_context` only for action-dependent behavior that cannot be expressed declaratively. Include explicit defaults and preserve `super()` output where extending behavior.

# Under the Hood

Generic mixins call hooks at specific points. `get_object` invokes queryset filtering and object permission checks; create mixin validates, calls `perform_create`, then builds response. Understanding this call chain prevents accidental bypass.

# Performance Notes

Avoid evaluating expensive querysets repeatedly in hooks. Cache only per-request values when safe; do not share evaluated querysets across requests.

# Best Practices

- Prefer `get_queryset` for scope, `perform_create` for save-time fields.
- Override action-level serializer selection explicitly.
- Use `super()` when extending framework behavior.
- Write tests that prove standard pagination/status/permission behavior remains.

# Common Mistakes

- Overriding `create` when `perform_create` is sufficient.
- Returning from a hook without expected result.
- Forgetting superclass behavior.
- Using request method strings where action metadata is available.

# Debugging

Set breakpoints/log safe identifiers at hook boundaries. Trace which mixin calls the method and inspect serializer/queryset/action values.

# Interview Questions

1. What does `perform_create` customize? Persistence save call after validation.
2. Why override `get_queryset`? Request-specific filtering and access scope.
3. Why prefer hooks over full handler override? Retain pagination, headers, and standard flow.
4. When call `super()`? When extending inherited behavior rather than replacing it.

# Practice Exercises

1. Assign owner through `perform_create`.
2. Scope a detail view through `get_queryset`.
3. Compare overriding `create` vs `perform_create` and test headers/status.

# Chapter Summary

DRF hooks are stable customization points in the generic lifecycle. Choose the narrowest hook, preserve inherited behavior when extending, and test the full endpoint after overrides.

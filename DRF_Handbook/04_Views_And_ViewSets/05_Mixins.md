# Introduction

DRF mixins provide reusable implementations for common model operations: list, create, retrieve, update, and destroy. They are designed to be combined with GenericAPIView.

# Why this concept exists

CRUD handlers repeat similar serializer, queryset, status, and response logic. Mixins share those mechanics while letting a view expose only selected HTTP methods.

# Real-world analogy

Mixins are modular tools in a repair kit: select only the tools needed for a particular job rather than carrying an entire machine.

# Internal Working

```text
HTTP handler -> mixin method (list/create/retrieve/update/destroy)
             -> GenericAPIView helpers -> serializer/queryset -> Response
```

# Request Lifecycle

APIView dispatch selects `get`/`post`/etc. The handler delegates to a mixin method, which uses GenericAPIView helpers and returns a response.

# Syntax

```python
class ItemList(mixins.ListModelMixin, generics.GenericAPIView):
    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)
```

# Complete Example 1

Build a read-and-create collection endpoint. Assume `Task` and `TaskSerializer` exist.

```python
# tasks/views.py
from rest_framework import generics, mixins
from .models import Task
from .serializers import TaskSerializer


class TaskCollection(
    mixins.ListModelMixin,
    mixins.CreateModelMixin,
    generics.GenericAPIView,
):
    serializer_class = TaskSerializer

    def get_queryset(self):
        return Task.objects.filter(owner=self.request.user).order_by("-id")

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

```python
# tasks/urls.py
from django.urls import path
from .views import TaskCollection

urlpatterns = [path("tasks/", TaskCollection.as_view(), name="task-list")]
```

```python
# settings.py (ensure policy or view permissions require authentication)
REST_FRAMEWORK = {
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
}
```

**Line-by-line:** List/Create mixins supply action implementations. GenericAPIView supplies serializer/queryset helpers. `get_queryset` limits records to the current user. GET delegates to `list`; POST delegates to `create`; `perform_create` injects owner. Settings establish a secure default.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** router -> handler -> mixin -> owner-filtered queryset or serializer validation -> ORM -> response. **Internal execution:** create mixin validates, calls `perform_create`, then constructs response; list mixin paginates if configured and serializes. **Professional use case:** Only expose list/create operations when update/delete are out of scope. Removing owner filtering risks cross-user disclosure; removing method handlers makes mixins unreachable.

# Complete Example 2

Detail endpoint with retrieve and delete:

```python
class TaskDetail(
    mixins.RetrieveModelMixin,
    mixins.DestroyModelMixin,
    generics.GenericAPIView,
):
    serializer_class = TaskSerializer

    def get_queryset(self):
        return Task.objects.filter(owner=self.request.user)

    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```

Map a URL with `<int:pk>`. `get_queryset` scopes `get_object`; retrieve returns representation; destroy returns the configured no-content response. Add authentication and test ownership.

# Complete Example 3

Update-only handler:

```python
class TaskUpdate(
    mixins.UpdateModelMixin,
    generics.GenericAPIView,
):
    ...

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def patch(self, request, *args, **kwargs):
        return self.partial_update(request, *args, **kwargs)
```

`update` handles full update; `partial_update` sets partial behavior. Full code includes the queryset/serializer and URL pattern as in previous examples. Do not expose methods not intended by product policy.

# Advanced Example

Mixins are usually ordered before GenericAPIView in inheritance lists to make method resolution clear. Understand Python MRO when combining custom mixins; avoid overriding a method without checking which parent implementation it replaces.

# Under the Hood

`ListModelMixin.list` filters, paginates, serializes, and returns a response. `CreateModelMixin.create` validates, saves via `perform_create`, sets success headers, and returns output. Retrieve/update/destroy similarly use `get_object`.

# Performance Notes

Mixin behavior can paginate but does not automatically optimize ORM relation queries. Customize `get_queryset` and serializer field strategy.

# Best Practices

- Include only needed mixins.
- Keep access scope in `get_queryset`.
- Override hooks such as `perform_create` rather than duplicating full mixin code.
- Test each exposed HTTP method and denied method.

# Common Mistakes

- Forgetting to wire method handlers.
- Adding all mixins without product need.
- Using unrestricted queryset for detail operations.
- Reimplementing mixin internals unnecessarily.

# Debugging

Trace HTTP method -> handler -> mixin method -> generic helper. Inspect queryset and serializer; confirm `get_object` returns expected 404/permission outcomes.

# Interview Questions

1. What is a mixin? Reusable class behavior composed with a view.
2. What base is commonly paired with CRUD mixins? GenericAPIView.
3. What does `perform_create` customize? The save step for create operations.
4. Are mixins security policy? No; configure authentication/permissions and scope data.

# Practice Exercises

1. Compose only list/create operations.
2. Add retrieve/destroy detail composition scoped by owner.
3. Compare code with `ListCreateAPIView`.

# Chapter Summary

Mixins expose reusable operation implementations that can be selectively composed with GenericAPIView. Use helper hooks, scope querysets, and avoid hand-rolling behavior the mixin already handles.

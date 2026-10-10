# Introduction

Concrete generic views combine GenericAPIView with common mixins to implement standard operations with little code. Examples include list/create, retrieve/update, and destroy endpoints.

# Why this concept exists

Most resource endpoints follow repeatable patterns. Concrete views let developers declare queryset, serializer, and policy without manually connecting each mixin method.

# Real-world analogy

A concrete generic view is a preassembled service counter for a known task; configure its data and rules instead of assembling every tool by hand.

# Internal Working

```text
ListCreateAPIView = GenericAPIView + ListModelMixin + CreateModelMixin
RetrieveUpdateAPIView = GenericAPIView + retrieve/update mixins
```

# Request Lifecycle

DRF APIView dispatch receives the method. The concrete view's inherited handler delegates to the matching mixin, which gets data/serializer helpers from GenericAPIView.

# Syntax

```python
class ItemListCreate(generics.ListCreateAPIView):
    queryset = Item.objects.all()
    serializer_class = ItemSerializer
```

# Complete Example 1

Authenticated users list and create their own tasks. Assume a `Task` model with `owner`, and `TaskSerializer`.

```python
# tasks/views.py
from rest_framework import generics
from rest_framework.permissions import IsAuthenticated
from .models import Task
from .serializers import TaskSerializer


class TaskListCreateView(generics.ListCreateAPIView):
    serializer_class = TaskSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Task.objects.filter(owner=self.request.user).order_by("-created_at")

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

```python
# tasks/urls.py
from django.urls import path
from .views import TaskListCreateView

urlpatterns = [path("tasks/", TaskListCreateView.as_view(), name="task-list")]
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("tasks.urls"))]
```

**Line-by-line:** The class inherits standard GET/POST behavior. `serializer_class` chooses data conversion. Authentication is required. `get_queryset` restricts list scope to caller and orders results; `perform_create` sets trusted owner after validation. URL registers collection.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** URL -> generic dispatch -> list or create mixin -> serializer -> owner-scoped ORM -> Response. **Internal execution:** POST validates before `perform_create`; GET serializes queryset. **Professional use case:** Standard per-user task collection. Removing `perform_create` may fail required owner constraint or allow client ownership if writable; removing queryset scope leaks records.

# Complete Example 2

Detail retrieve/update:

```python
class TaskDetailView(generics.RetrieveUpdateAPIView):
    serializer_class = TaskSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Task.objects.filter(owner=self.request.user)
```

With URL `tasks/<int:pk>/`, GET retrieves, PUT updates fully, PATCH updates partially. `get_queryset` ensures lookup itself is scoped. `RetrieveUpdateAPIView` does not include delete.

# Complete Example 3

Read-only list and detail:

```python
class TaskReadOnlyView(generics.RetrieveAPIView):
    queryset = Task.objects.filter(is_public=True)
    serializer_class = TaskSerializer
```

Add a separate `ListAPIView` for collection. This intentionally avoids create/update methods. A list and a detail route have different lookup configuration; do not assume one class automatically exposes both.

# Advanced Example

Choose `ListCreateAPIView`, `RetrieveUpdateDestroyAPIView`, and related classes by exact supported method set. Use `get_serializer_class()` if list and detail representations differ, but test each action and contract.

# Under the Hood

Concrete views are compositions of GenericAPIView and mixins. They inherit lifecycle handlers such as `get`/`post`; their methods call mixin operations that use `get_queryset`, `get_object`, pagination, serializer, and save hooks.

# Performance Notes

Use pagination for lists and relation optimization for serializers. A class-level queryset should not be evaluated and cached manually across requests.

# Best Practices

- Use a concrete view for standard CRUD slices.
- Scope `get_queryset` by request/user.
- Override `perform_create` for server-owned values.
- Keep only supported methods and test them.

# Common Mistakes

- Assuming one generic class supports every operation.
- Forgetting queryset ownership filtering.
- Using `queryset = Model.objects.all()` for private records.
- Duplicating mixin behavior in overridden handlers.

# Debugging

Check inherited method set, URL kwargs, `get_queryset`, permission result, serializer class, and response status. Test unsupported methods too.

# Interview Questions

1. What is a concrete generic view? A precomposed GenericAPIView plus standard mixin behavior.
2. What operations does ListCreateAPIView support? GET collection and POST create.
3. Why override `get_queryset`? Dynamic scoping/filtering.
4. What does `perform_create` do? Customize save-time values.

# Practice Exercises

1. Build list/create and detail/update/destroy endpoints with owner scope.
2. Add pagination to list.
3. Test each exposed and rejected method.

# Chapter Summary

Concrete generic views are a concise choice for conventional endpoint operations. Select the precise class, scope querysets securely, and customize hooks rather than rebuilding its lifecycle.

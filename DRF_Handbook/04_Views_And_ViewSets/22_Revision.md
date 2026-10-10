# Introduction

This revision gathers the essential DRF view concepts: function views, APIView, generic views, mixins, ViewSets, routers, hooks, lifecycle, and optimization.

# Why this concept exists

Views are where routing, policy, serializers, and data access meet. A compact review helps select the right abstraction and debug requests systematically.

# Real-world analogy

Different view classes are service-desk designs: a single custom desk, a general counter, prebuilt CRUD service, or a resource department with a generated directory.

# Internal Working

```text
URL/router -> dispatch/action -> authentication/permissions
           -> parser -> serializer/query -> DB/service -> Response -> renderer
```

# Request Lifecycle

1. Django resolves URL; router maps method to ViewSet action when applicable.
2. DRF initializes Request and content negotiation.
3. Authentication, permissions, and throttles execute.
4. Handler/mixin/action processes request.
5. Parser produces body data when accessed; serializer validates/represents.
6. ORM/service reads or writes.
7. DRF handles exceptions and renders response.

# Syntax

```python
class TaskViewSet(viewsets.ModelViewSet):
    serializer_class = TaskSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Task.objects.filter(owner=self.request.user)
```

# Complete Example 1

Minimal user-scoped read endpoint:

```python
# tasks/views.py
from rest_framework import generics
from rest_framework.permissions import IsAuthenticated
from .models import Task
from .serializers import TaskSerializer


class TaskListView(generics.ListAPIView):
    serializer_class = TaskSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Task.objects.filter(owner=self.request.user).order_by("-id")
```

```python
# tasks/urls.py
from django.urls import path
from .views import TaskListView

urlpatterns = [path("tasks/", TaskListView.as_view(), name="task-list")]
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("tasks.urls"))]
```

**Line-by-line:** ListAPIView supplies GET list flow. Serializer class maps output. Authentication is required. `get_queryset` scopes and orders. URL connects the view and project include.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** URL resolver -> generic handler -> permission -> scoped queryset -> serializer -> renderer. No write occurs. **Internal execution:** The generic view paginates if configured. **Professional tip:** For create, assign owner through `perform_create`; do not accept it from client. Removing scope risks leakage.

# Complete Example 2

Minimal router registration:

```python
router = SimpleRouter()
router.register("tasks", TaskViewSet, basename="task")
urlpatterns = router.urls
```

This generates route/action maps for ViewSet actions. It does not add permission policy or data scoping.

# Complete Example 3

Select abstraction:

| Need | Good starting point |
|---|---|
| Tiny endpoint | `@api_view` |
| Custom method logic | `APIView` |
| Reusable resource mechanics | `GenericAPIView` + mixins |
| Conventional operation set | Concrete generic view |
| Grouped resource actions | ViewSet + router |

# Advanced Example

ViewSet custom action:

```python
@action(detail=True, methods=["post"])
def archive(self, request, pk=None):
    obj = self.get_object()
    ...
```

Use action metadata and scoped object lookup; validate payload and transition rules; wrap related writes transactionally.

# Under the Hood

APIView dispatch is the common base. GenericAPIView adds queryset/serializer helpers. Mixins implement actions. Routers bind HTTP methods to ViewSet action methods.

# Performance Notes

Paginate; scope/filter in SQL; optimize relation loads; avoid N+1 serializer methods; measure query counts.

# Best Practices

- Choose minimal fitting abstraction.
- Scope querysets and configure permissions.
- Use serializers and server-owned values.
- Preserve inherited behavior through hooks.
- Test routes, methods, error paths, and isolation.

# Common Mistakes

- Assuming router or ModelViewSet secures endpoints.
- Exposing unnecessary CRUD actions.
- Bypassing `get_object`/object permission checks.
- Rewriting generic handlers for small customization.

# Debugging

Trace URL -> action/method -> policy -> parser -> serializer -> queryset/database -> response. Inspect generated routes and test using APIClient.

# Interview Questions

1. Why choose GenericAPIView? Reusable resource helpers and hooks.
2. What does router add? URL patterns/action mappings.
3. Why override `get_queryset`? Request-dependent visibility/scope.
4. How avoid N+1? Query optimization aligned with serializer access.

# Practice Exercises

1. Explain FBV, APIView, generic view, and ViewSet differences.
2. Scope a ModelViewSet to user-owned records.
3. Add custom action and test permissions.

# Chapter Summary

All view abstractions use DRF's request lifecycle; they differ in handler organization and reuse. Understand route/action mapping, policy checks, queryset scope, serializer work, and response finalization before choosing one.

# Introduction

`ModelViewSet` combines generic queryset/serializer behavior with list, retrieve, create, update, partial update, and destroy actions. It is a concise option for standard model CRUD.

# Why this concept exists

A resource commonly needs a standard set of operations. ModelViewSet removes repetitive method wiring while preserving serializer, queryset, permission, and hook customization.

# Real-world analogy

ModelViewSet is a fully equipped service counter for one database resource, with a standard window for each CRUD action.

# Internal Working

```text
router -> method/action map -> ModelViewSet action
 -> mixin -> queryset/object + serializer -> ORM -> response
```

# Request Lifecycle

The router maps collection/detail paths and HTTP methods to actions. DRF applies policies; mixins perform list/retrieve/create/update/delete using GenericAPIView helpers.

# Syntax

```python
class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
```

# Complete Example 1

Authenticated users can manage only their own tasks. Assume `Task` has `owner` and `created_at`, and the serializer exposes safe fields.

```python
# tasks/views.py
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticated
from .models import Task
from .serializers import TaskSerializer


class TaskViewSet(viewsets.ModelViewSet):
    serializer_class = TaskSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Task.objects.filter(owner=self.request.user).order_by("-created_at")

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

```python
# tasks/urls.py
from rest_framework.routers import SimpleRouter
from .views import TaskViewSet

router = SimpleRouter()
router.register("tasks", TaskViewSet, basename="task")
urlpatterns = router.urls
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("tasks.urls"))]
```

**Line-by-line:** ModelViewSet supplies six standard actions. Serializer defines fields/validation. Authentication is required. `get_queryset` scopes every lookup/list to the current user. `perform_create` assigns trusted ownership. Router creates routes with explicit basename. Project includes them.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** router maps method/path -> action -> permission -> queryset/serializer -> ORM mutation/read -> response. **Internal execution:** `partial_update` handles PATCH; `update` handles PUT; detail actions use `get_object()` and object permissions. **Professional use case:** Standard private CRUD resource. Removing owner scope can expose other users' tasks; removing `perform_create` may break ownership integrity.

# Complete Example 2

Disable destroy if deletion is not supported:

```python
class TaskViewSet(
    mixins.ListModelMixin,
    mixins.RetrieveModelMixin,
    mixins.CreateModelMixin,
    viewsets.GenericViewSet,
):
    ...
```

Only these actions are composed, so router does not map update/destroy handlers. This is safer than exposing all actions and later hiding buttons in a client.

# Complete Example 3

Choose a serializer by action:

```python
def get_serializer_class(self):
    if self.action == "list":
        return TaskSummarySerializer
    if self.action in {"create", "update", "partial_update"}:
        return TaskWriteSerializer
    return TaskDetailSerializer
```

All serializers must be defined and compatible with each operation. The action is set by router dispatch. Test each action's request and response schema.

# Advanced Example

Use `get_permissions()` for action-specific policy only when requirements differ, and make the fallback explicit. Review custom actions separately; `ModelViewSet` standard behavior does not define business-specific permissions.

# Under the Hood

ModelViewSet inherits GenericViewSet and CRUD mixins. Router maps HTTP verbs to action methods; generic helpers obtain serializer, queryset, object, pagination, and save hooks.

# Performance Notes

Paginate list actions; optimize relation access; avoid expensive serializer method fields. A single ModelViewSet class can still generate slow queries if `get_queryset` is not designed well.

# Best Practices

- Use ModelViewSet only when its full action set is appropriate.
- Scope `get_queryset` to caller/data visibility.
- Set server-owned fields in perform hooks.
- Enumerate serializer fields and test every action.

# Common Mistakes

- Exposing DELETE/PUT without product intent.
- Using unrestricted queryset for private models.
- Trusting client ownership.
- Assuming router paths imply permission checks.

# Debugging

Inspect router route/action map, `self.action`, effective permissions, serializer class, queryset, and response codes. Test 404 for objects outside scoped queryset.

# Interview Questions

1. What actions come with ModelViewSet? List, retrieve, create, update, partial_update, destroy.
2. How restrict actions? Compose GenericViewSet with selected mixins.
3. Why `get_queryset`? Per-request filtering and secure visibility.
4. What sets owner on create? `perform_create` or explicit service logic.

# Practice Exercises

1. Build a user-scoped Task ModelViewSet.
2. Restrict it to list/retrieve/create.
3. Use separate summary/write serializers.

# Chapter Summary

ModelViewSet efficiently exposes conventional model CRUD when that complete surface is appropriate. Router wiring is not authorization; scope querysets and configure serializers/policies deliberately.

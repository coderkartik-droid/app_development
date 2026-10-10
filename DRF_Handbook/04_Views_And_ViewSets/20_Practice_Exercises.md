# Introduction

These exercises progress from method dispatch to resource routing, permissions, customization, and performance. Implement them in a Django project with DRF installed.

# Why this concept exists

Views are best learned by tracing actual HTTP methods through routes, actions, serializers, querysets, and responses—not by memorizing class names alone.

# Real-world analogy

Practice is a simulation room: safely rehearse every operation and failure path before exposing a real endpoint.

# Internal Working

```text
route/action -> dispatch -> policy -> serializer/queryset -> DB -> Response
```

# Request Lifecycle

For each exercise, identify the stage where method mapping, authentication, parsing, validation, persistence, and rendering occur.

# Syntax

```python
class ItemList(generics.ListCreateAPIView):
    serializer_class = ItemSerializer
```

# Complete Example 1

Create a status endpoint in an installed `api` app:

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["GET"])
def status_view(request):
    return Response({"status": "exercise"})
```

```python
# api/urls.py
from django.urls import path
from .views import status_view

urlpatterns = [path("status/", status_view, name="status")]
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("api.urls"))]
```

**Line-by-line:** The decorator restricts methods; view returns primitive data; app URL maps endpoint; project URL includes it. **Request → Router → View/ViewSet → Serializer → Database → Response flow:** route -> function -> no serializer/database in simple status example -> Response. **Internal:** DRF finalizes response. **Professional use case:** Service readiness route, not a substitute for detailed observability.

# Complete Example 2

Exercise: compose a user-owned task endpoint:

```python
class TaskCollection(generics.ListCreateAPIView):
    serializer_class = TaskSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Task.objects.filter(owner=self.request.user)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

Add a URL, test GET/POST, and prove a second user cannot see the first user's tasks.

# Complete Example 3

Exercise: router and ViewSet:

```python
router = SimpleRouter()
router.register("tasks", TaskViewSet, basename="task")
urlpatterns = router.urls
```

Write tests for list, detail, create, update, delete, custom action, and unsupported method. Inspect generated route names and action map.

# Advanced Example

Optimize the nested task list using appropriate relation loading and pagination. Measure query count before and after, then add a test guarding against N+1 regressions.

# Under the Hood

Test client calls pass through Django routing and DRF dispatch without a live server. API tests should validate not only handler output but permissions, statuses, and database effects.

# Performance Notes

Use representative row counts and nested relations. Query-count assertions are most useful when they protect an important bound, not every incidental query.

# Best Practices

- Use temporary test data and isolated users.
- Test success and denial branches.
- Verify no database side effects on invalid input.
- Assert router-generated paths by reversing names.

# Common Mistakes

- Testing views by direct function call only.
- Forgetting route inclusion or basename.
- Ignoring cross-user tests.
- Using global in-memory lists shared across tests.

# Debugging

Check URL reverse/resolution, method, action, permission, queryset, serializer, DB state, and response. Read failing test output before changing implementation.

# Interview Questions

1. Why use APIClient? Exercise HTTP dispatch and DRF behavior.
2. What is a useful cross-user test? User A's data is absent/inaccessible to user B.
3. How test unsupported methods? Assert 405 and no side effect.
4. Why count queries? Catch N+1/performance regressions.

# Practice Exercises

1. Complete status endpoint and 405 test.
2. Implement user-owned generic view and isolation tests.
3. Build routed ViewSet and action test.
4. Optimize nested list and measure query count.

# Chapter Summary

Practice the full view path: route/action, dispatch, policy, serializer, queryset, persistence, and response. Tests should prove both client contract and data isolation.

# Introduction

The DRF request lifecycle is the ordered path from incoming HTTP bytes to an outgoing response. Knowing this path helps place validation, permissions, database access, and rendering in the correct layer.

# Why this concept exists

When a request fails, the cause may be routing, method dispatch, content negotiation, authentication, parsing, validation, persistence, or rendering. A lifecycle model narrows debugging.

# Real-world analogy

A package moves through reception, access checks, unpacking, inspection, processing, packing, and dispatch. Each station handles a different responsibility.

# Internal Working

```text
HTTP bytes
 -> Django middleware
 -> URL resolver/router action map
 -> DRF initialize_request + content negotiation
 -> authentication -> permissions -> throttles
 -> parser on request.data access
 -> handler/action -> serializer validation -> ORM/service
 -> Response -> exception/finalization -> renderer -> HTTP bytes
```

# Request Lifecycle

1. Django receives request and runs request middleware.
2. URL resolver selects a view; routers may map HTTP method to ViewSet action.
3. DRF wraps `HttpRequest` as `Request` and negotiates renderer.
4. Authentication, permissions, and throttles run.
5. Handler accesses query/body data; parser is selected lazily.
6. Serializer validates; application logic queries or writes data.
7. DRF handles known exceptions and finalizes `Response`.
8. Renderer turns primitive data into response bytes; response middleware runs.

# Syntax

```python
serializer = ItemSerializer(data=request.data)
serializer.is_valid(raise_exception=True)
item = serializer.save()
return Response(ItemSerializer(item).data, status=201)
```

# Complete Example 1

This complete ModelViewSet flow assumes `Task`, `TaskSerializer`, and installed app configuration.

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
        return Task.objects.filter(owner=self.request.user)

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

**Line-by-line:** ModelViewSet supplies CRUD actions. Serializer controls fields/validation. Authentication gates access. Queryset restricts objects; create hook injects owner. Router maps methods/routes; project mounts routes.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** URL method map -> action -> authentication/permission -> parser -> serializer -> owner-scoped ORM -> serializer representation -> renderer. **Internal working:** Detail lookup uses scoped queryset, so an inaccessible ID behaves as not found. **Professional use case:** User-owned task CRUD. Removing query scoping leaks other users' data; removing authentication may expose operations depending on defaults.

# Complete Example 2

Trace malformed JSON: request arrives with a JSON media type but invalid body; parser raises a parsing exception before serializer validation, and DRF returns 400. No database write should occur. This distinction matters: parser errors mean bytes could not be decoded; serializer errors mean decoded values violated the data contract.

# Complete Example 3

Trace unacceptable `Accept`: negotiation happens before action logic; if no renderer matches, DRF returns 406 without querying the resource. Test this when renderer formats are restricted.

# Advanced Example

Middleware is Django-wide and runs around the view. DRF authentication is endpoint-policy processing after API request initialization; do not assume middleware and authentication classes are interchangeable.

# Under the Hood

APIView dispatch calls request initialization, content negotiation, `initial()` policy checks, method/action handler, exception handling, and response finalization. Request body parsing and queryset evaluation are often lazy.

# Performance Notes

Parsing, authentication, ORM execution, serializer iteration, and rendering all contribute. Time each stage before optimizing.

# Best Practices

- Place validation in serializers and access policy in permissions/querysets.
- Keep writes transactional when invariants span records.
- Test failure at every lifecycle stage.
- Avoid side effects before validation/authorization.

# Common Mistakes

- Assuming parser and serializer errors are the same.
- Performing database writes before permission checks.
- Debugging only view code while route/negotiation fails earlier.
- Expecting response data to be rendered before finalization.

# Debugging

Check in order: URL/action, method, negotiation, authentication, permissions, parser, serializer, queryset/DB, response renderer. Find the first stage not reached.

# Interview Questions

1. When does authentication run? During API view initialization/policy checks before handler.
2. When parser runs? Usually when `request.data` is accessed.
3. When response renders? During finalization after handler return.
4. What happens before handler on permission denial? DRF returns/raises a permission exception.

# Practice Exercises

1. Draw lifecycle for POST create and annotate where each class runs.
2. Create one parser error and one serializer validation error; compare.
3. Test 406 and 401/403 behavior.

# Chapter Summary

DRF's lifecycle moves through routing, negotiation, policy, parsing, handler, serialization/persistence, and rendering. Debug in order and keep each concern at its proper stage.

# Introduction

DRF views connect HTTP requests to application behavior and responses. The view family ranges from function-based views for small explicit endpoints to APIView, generic views, and ViewSets for increasing levels of reuse.

# Why this concept exists

An API needs a place to coordinate permissions, input handling, serializers, data access, and response status. Different view abstractions let teams choose how much behavior to reuse.

# Real-world analogy

Views are service desks. A small desk handles one task itself; a standard counter follows a reusable procedure; a ViewSet groups related operations under one resource.

# Internal Working

```text
URL/router -> view callable -> DRF dispatch -> policy checks -> handler/action
           -> serializer -> ORM -> Response -> renderer
```

Views do not replace serializers, model constraints, or authorization design. They orchestrate these pieces.

# Request Lifecycle

Django resolves the URL, then DRF initializes the request, negotiates a renderer, runs configured authentication/permissions/throttles, dispatches to a method or action, handles exceptions, and finalizes the response.

# Syntax

```python
class ItemList(generics.ListAPIView):
    queryset = Item.objects.all()
    serializer_class = ItemSerializer
```

# Complete Example 1

This minimal DRF function-based view returns a health response. Assume Django and DRF are installed and `"rest_framework"` is in `INSTALLED_APPS`.

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["GET"])
def health(request):
    return Response({"status": "ok"})
```

```python
# api/urls.py
from django.urls import path
from .views import health

urlpatterns = [path("health/", health, name="health")]
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("api.urls"))]
```

**Line-by-line:** `api_view` adapts a function to DRF's dispatch pipeline and lists permitted methods. `Response` carries primitive data until rendering. App URL maps the endpoint; project URL mounts the app. **Flow:** request -> router/URL resolver -> function wrapper -> health function -> Response -> renderer. **Internal execution:** Unsupported methods are rejected before the function body. **Professional use case:** Small, isolated endpoints. Removing the decorator leaves a plain Django function without DRF request/response processing.

# Complete Example 2

A class-based API view:

```python
class HealthView(APIView):
    def get(self, request):
        return Response({"status": "ok"})
```

Register with `path("health/", HealthView.as_view())`. APIView supports policy attributes and multiple method handlers. Use it when function decorators become crowded or class configuration is useful.

# Complete Example 3

A generic list endpoint:

```python
class ItemList(generics.ListAPIView):
    queryset = Item.objects.all()
    serializer_class = ItemSerializer
```

The generic view supplies standard list behavior. Use it when the resource operation follows a conventional pattern; override `get_queryset` for request-specific scoping.

# Advanced Example

A ViewSet groups list/retrieve/create/update/destroy actions and can be routed by a router. It reduces repeated URL wiring but requires understanding action mapping and permission behavior.

# Under the Hood

DRF views are Django-compatible callables. APIView's dispatch wraps request processing and response finalization. Generic views combine APIView and mixins; ViewSets expose actions that routers map to HTTP methods.

# Performance Notes

The abstraction level rarely dominates latency. Optimize querysets, serializer relation access, payload size, and pagination.

# Best Practices

- Choose the simplest view abstraction that fits.
- Scope querysets to the caller.
- Use serializers for input/output contracts.
- Apply permissions explicitly and test HTTP methods.

# Common Mistakes

- Assuming class-based views automatically secure resources.
- Returning raw models or querysets.
- Using a ViewSet without understanding its generated routes.
- Overriding a framework method without calling required behavior.

# Debugging

Trace route match, method/action mapping, policy checks, serializer, queryset, and response. Inspect `request.resolver_match`, view configuration, status, and test results.

# Interview Questions

1. When choose FBV? Small explicit endpoint.
2. What does APIView add? DRF request/response dispatch and policy hooks.
3. Why generic view? Reuse common CRUD behavior.
4. What does ViewSet group? Related resource actions.

# Practice Exercises

1. Implement one status route as FBV and APIView.
2. Implement a generic list endpoint.
3. Draw the request path for each abstraction.

# Chapter Summary

DRF view abstractions share the same request lifecycle but differ in reuse and organization. Choose deliberately, preserve clear serializer/data boundaries, and enforce access at the server.

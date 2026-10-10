# Introduction

Django REST Framework (DRF) is a toolkit for building Web APIs with Django. It adds API-oriented request parsing, serializers, authentication and permission hooks, content negotiation, browsable API pages, and reusable views.

An API usually returns structured data such as JSON rather than server-rendered HTML. DRF still uses Django's project settings, URL routing, models, and database layer.

# Why this concept exists

Plain Django can return JSON, but a growing API also needs consistent input validation, output representation, status handling, permissions, and tests. DRF provides reusable components for those responsibilities rather than making every view reinvent them.

# Real-world analogy

Django is the building and its routing system; DRF is a staffed service counter that understands API forms, checks access rules, validates submissions, and returns standardized receipts.

# Internal Working

```text
HTTP request -> Django middleware -> URL resolver -> DRF APIView
             -> authentication -> permissions -> parser -> handler
             -> serializer/ORM -> Response -> renderer -> HTTP response
```

DRF's `APIView` adapts Django's request/response lifecycle. It wraps the incoming Django request in DRF's `Request`, applies configured policies, dispatches to a method handler, and finalizes a DRF `Response` through a renderer.

# Request Lifecycle (where applicable)

For a JSON `POST`, Django receives bytes and headers. DRF's parser decodes the body when `request.data` is accessed. A serializer validates the resulting Python values. The view may save data using the ORM, then returns a `Response`; content negotiation selects a renderer, commonly JSON.

# Syntax

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["GET"])
def health(request):
    return Response({"status": "ok"})
```

# Complete Example 1

Create a Django project and app, install `djangorestframework`, add `"rest_framework"` and `"api"` to `INSTALLED_APPS`, then use the following files. The route returns a JSON representation without a database.

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["GET"])
def hello_api(request):
    return Response({"message": "Hello from DRF"})
```

```python
# api/urls.py
from django.urls import path
from .views import hello_api

urlpatterns = [path("hello/", hello_api, name="hello-api")]
```

```python
# config/urls.py
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", include("api.urls")),
]
```

Run `python manage.py runserver` and request `/api/hello/`.

**Line-by-line:** `api_view(["GET"])` adapts a function into a DRF endpoint and limits allowed methods. `Response` stores Python primitives until finalization. The app URLconf maps `hello/`; the project URLconf mounts it under `api/`. **Execution flow:** client request -> Django URL resolver -> DRF wrapper -> GET handler -> DRF `Response` -> JSON renderer -> response bytes. **Internal working:** This view has no model or serializer because its output is a constant. **Professional explanation:** Use this simple endpoint to verify project wiring, but put resource validation and data access behind appropriate layers as the API grows. Removing `@api_view` leaves a plain Django function that does not get DRF's `Request`, method checks, or response finalization.

# Complete Example 2

Return a collection of primitive records:

```python
@api_view(["GET"])
def colors(request):
    return Response({"results": [{"name": "blue"}, {"name": "green"}]})
```

The outer mapping provides a stable response envelope; the list represents results. DRF's renderer serializes the Python values. Without `Response`, returning a dictionary directly is not a valid Django view response. **Flow:** handler data -> `Response` -> renderer -> JSON. **Tip:** For model-backed data, use a serializer rather than assembling complex output ad hoc.

# Complete Example 3

Reject unsupported methods naturally:

```python
@api_view(["GET"])
def read_only_status(request):
    return Response({"read_only": True})
```

DRF returns `405 Method Not Allowed` for unsupported methods and includes an `Allow` header. If the decorator is removed, this API-specific method restriction disappears. Test both allowed and disallowed verbs.

# Advanced Example

Set API-wide defaults in `REST_FRAMEWORK` only after deciding the security policy. Authentication identifies a caller; permissions decide access; throttling can limit request rates. These policies run around endpoint logic and should be tested at both global and per-view scope.

# Under the Hood

`APIView` and `@api_view` use DRF's dispatch/finalization pipeline. DRF exceptions are converted into API responses by its exception handling; unhandled programming errors still need correct logging and should not leak tracebacks to clients.

# Performance Notes

Do not load large collections without pagination. Avoid repeated ORM access in loops; query related objects efficiently and serialize only fields clients need.

# Best Practices

- Keep endpoints resource-oriented and document request/response shapes.
- Validate all client input.
- Choose explicit permissions; never rely on hidden UI controls.
- Return meaningful HTTP status codes.
- Test success, invalid input, unauthorized access, and unsupported methods.

# Common Mistakes

- Confusing `Response` with Django's `HttpResponse`.
- Returning ORM objects directly.
- Assuming `@api_view` creates authentication policy automatically.
- Exposing all model fields without reviewing privacy.

# Debugging

Check the mounted URL, installed app, DRF settings, accepted method, parser/renderers, and response status. Use DRF's browsable API in development and inspect logs for server-side failures.

# Interview Questions

1. What does DRF add to Django? API views, serializers, parsing/rendering, authentication, permissions, and related utilities.
2. What does `Response` do? Holds unrendered API data for renderer selection.
3. Where is the database accessed? Commonly through Django's ORM in a view/service/repository boundary.
4. Does DRF automatically secure every endpoint? No; policy must be configured and tested.

# Practice Exercises

1. Add a `/api/version/` GET route with a JSON object.
2. Test that POST to the GET-only endpoint returns 405.
3. Inspect the response content type and status.

# Chapter Summary

DRF builds APIs on Django's request, routing, and ORM foundations. Its pipeline parses requests, applies policy, invokes a handler, and renders a `Response`. Begin with clear URL wiring and add serializers, data access, and security intentionally.

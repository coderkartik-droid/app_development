# Introduction

A function-based view (FBV) is a Python function that accepts an `HttpRequest` and returns an `HttpResponse`. It is the most direct way to see Django's request/response contract.

# Why this concept exists

Functions make simple route behavior explicit and easy to read. They are useful for focused handlers with limited branching.

# Real-world analogy

An FBV is a recipe: receive ingredients (request), perform steps, and return the finished result (response).

# Internal Working

The URL resolver calls the function with the request and any captured route arguments. The function may query models, validate input, render a template, or return JSON.

```text
path match -> function(request, captured args) -> response
```

# Syntax

```python
def home(request):
    return HttpResponse("Hello")
```

# Complete Example 1

Create a JSON status FBV and route:

```python
# pages/views.py
from django.http import JsonResponse


def status(request):
    return JsonResponse({"status": "ready"})
```

```python
# pages/urls.py
from django.urls import path
from .views import status

urlpatterns = [path("status/", status, name="status")]
```

**Line-by-line:** Import `JsonResponse`; define the request handler; return a dictionary encoded as JSON; connect the route name to the function. **Flow:** HTTP request -> route match -> `status(request)` -> JSON response. **Internal:** Django's plain request exposes form/query data and headers; DRF adds convenient parsed request data and API policy. **Professional tip:** Keep this function small and use DRF for production API validation/authentication patterns.

# Complete Example 2

Handle methods explicitly:

```python
from django.http import HttpResponseNotAllowed, JsonResponse


def ping(request):
    if request.method != "GET":
        return HttpResponseNotAllowed(["GET"])
    return JsonResponse({"pong": True})
```

Only GET is accepted; other methods receive 405. **Flow:** URL resolves -> method check -> response branch. **Internal:** Correct HTTP method semantics help clients and intermediaries understand behavior. **Tip:** Django decorators and DRF views can express method restrictions more cleanly as behavior grows.

# Advanced Example

Use a DRF function-based view decorator such as `@api_view(["GET"])` in DRF chapters. It wraps Django's request lifecycle with API request parsing, response negotiation, and DRF exception handling; do not confuse it with an undecorated Django FBV.

# Common Mistakes

- Returning a Python dictionary instead of an `HttpResponse`/response object.
- Ignoring method semantics.
- Accessing raw data without validation.
- Writing a large function with unrelated operations.

# Best Practices

- Keep FBVs focused and direct.
- Return accurate status codes.
- Validate untrusted inputs and apply access rules.
- Use DRF decorators/classes when implementing API endpoints.

# Interview Questions

1. What does an FBV receive? Request and captured URL arguments.
2. What must it return? An HTTP response object.
3. How reject unsupported methods? Return/raise a method-not-allowed response.
4. What does DRF add? API parsing, responses, serializers, permissions, and related tools.

# Practice

Write a GET-only JSON view with a typed path parameter and return 404 for unknown records.

# Summary

FBVs provide a transparent request-to-response path. Keep them focused, validate input, and use DRF's API-aware view tools when building APIs.

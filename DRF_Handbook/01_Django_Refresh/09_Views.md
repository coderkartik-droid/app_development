# Introduction

A Django view is a callable that receives an HTTP request and returns an HTTP response. It contains request-specific coordination, such as selecting data or deciding which response to return.

# Why this concept exists

The URL resolver needs a function or class to handle a matched route. Views centralize behavior for that route.

# Real-world analogy

A view is a receptionist who receives a visitor's request, asks the right department for information, and returns a result.

# Internal Working

Middleware processes the request before the view. Django resolves the URL and calls the view with a request (and captured parameters). The view returns a response; middleware then processes the response.

```text
request -> middleware -> URL resolver -> view -> HttpResponse -> middleware -> client
```

# Syntax

```python
def detail(request, item_id):
    return HttpResponse(f"Item {item_id}")
```

# Complete Example 1

This function-based view returns a small JSON response. Add a URL pattern pointing to `status`.

```python
# catalog/views.py
from django.http import JsonResponse


def status(request):
    return JsonResponse({"service": "catalog", "ready": True})
```

```python
# catalog/urls.py
from django.urls import path
from .views import status

urlpatterns = [path("status/", status, name="status")]
```

**Line-by-line:** Import `JsonResponse`; define a callable with the request parameter; return a dictionary encoded as JSON. The URL pattern maps `status/` to the callable. **Flow:** request -> URL resolver -> view -> serialization -> response. **Internal:** Django invokes the view after middleware and route matching. **Professional tip:** Keep response behavior explicit and avoid exposing sensitive operational details.

# Complete Example 2

Render an HTML template:

```python
from django.shortcuts import render


def welcome(request):
    return render(request, "welcome.html", {"name": "Developer"})
```

The `request` provides context for rendering; the template path selects a template; the dictionary provides values. Connect the view to a URL and create the template. **Flow:** resolve -> build context -> render template -> return HTML. **Tip:** For APIs, DRF views and serializers are preferable to hand-building JSON for complex resources.

# Advanced Example

Return an appropriate status code:

```python
from django.http import HttpResponse


def created(request):
    return HttpResponse("Created", status=201)
```

The status is part of the HTTP response contract. **Internal:** Clients interpret status independently of body content. **Tip:** Use status codes consistently and avoid returning 200 for every outcome.

# Common Mistakes

- Forgetting to return a response.
- Performing unrelated business logic in a view.
- Returning the wrong status code.
- Assuming a Django view automatically validates API input.

# Best Practices

- Keep views focused on request/response coordination.
- Delegate data access and validation to appropriate layers.
- Use named routes and explicit response behavior.
- Test method, status, and response body.

# Interview Questions

1. What is a Django view? A request handler returning a response.
2. What input does a view receive? An `HttpRequest`, plus captured URL arguments.
3. What wraps the view? Request/response middleware.
4. Do plain Django views automatically provide DRF parsing? No.

# Practice

Write a view returning a 404 response for an unknown ID and connect it to a typed URL pattern.

# Summary

Views coordinate request-specific behavior and return responses. DRF extends this concept with API request parsing, serialization, permissions, and response handling.

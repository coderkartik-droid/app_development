# Introduction

Django is a Python web framework for building server-side applications. It supplies URL routing, request/response handling, templates, an ORM, authentication utilities, and an admin interface. This refresher covers the parts needed to understand Django REST Framework (DRF), not every Django feature.

# Why this concept exists

Web applications need consistent ways to receive requests, run logic, access data, and return a response. Django provides these common pieces so teams can focus on application behavior.

# Real-world analogy

Django is a well-organized workshop: URL routes direct jobs, views perform work, models describe stored records, and templates assemble a human-readable response.

# Internal Working

```text
HTTP request -> URL resolver -> view -> optional ORM/template -> response
```

Django's server receives a request, matches its path against URL patterns, invokes a view, then returns the view's `HttpResponse` (or a response produced by a helper). DRF builds on this lifecycle to return API representations.

# Syntax

```python
from django.http import HttpResponse

def hello(request):
    return HttpResponse("Hello")
```

# Complete Example 1

Create a Django project and add this view and URL configuration. In a terminal: `python -m pip install django`, `django-admin startproject config .`. This example's complete app code follows.

```python
# pages/views.py
from django.http import HttpResponse


def home(request):
    return HttpResponse("Django is ready")
```

```python
# config/urls.py
from django.contrib import admin
from django.urls import path
from pages.views import home

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", home, name="home"),
]
```

Create the app with `python manage.py startapp pages`, then run `python manage.py runserver`.

**Line-by-line:** Import `HttpResponse`; define `home` accepting the request; return text. In URL config, import `path` and the view; map the empty path to the view and name the route. Django resolves `/`, calls `home`, and sends the returned response. The request middleware runs before the view, and response middleware runs on the way out. **Professional tip:** Give named routes and keep URL configuration separate from business logic.

# Complete Example 2

```python
from django.http import JsonResponse


def health(request):
    return JsonResponse({"status": "ok"})
```

`JsonResponse` serializes a mapping to JSON and sets a JSON content type. The execution path is request -> URL resolver -> `health` -> JSON encoding -> response. The view is runnable when connected to a URL in an installed Django project. **Internal:** This is still ordinary Django; DRF later adds serializers, request parsing, and API-specific responses. **Tip:** Avoid exposing detailed internal diagnostics in a public health endpoint.

# Advanced Example

Use the named route from another view/template with `reverse("home")` or `{% url 'home' %}` rather than hard-coding paths. URL names allow route changes without updating every caller. **Execution:** Django resolves the name from the URLconf, builds the path, and the caller uses the result. **Tip:** Names should express purpose and remain unique within their namespace.

# Common Mistakes

- Confusing Django's HTML response with a DRF API response.
- Putting all application behavior directly in URL configuration.
- Forgetting to register an app in `INSTALLED_APPS`.
- Treating development server settings as production configuration.

# Best Practices

- Keep URL routing, view behavior, data models, and presentation distinct.
- Use Django's settings and management commands rather than hand-building framework internals.
- Learn request/response flow before adding DRF abstractions.

# Interview Questions

1. What is Django? A Python framework for server-side web applications.
2. What does a view do? Processes a request and returns a response.
3. What routes a request to a view? URL patterns and Django's URL resolver.
4. How does DRF relate to Django? It adds tools for building web APIs on top of Django.

# Practice

Create a `status/` route returning `{"status": "ready"}`. Name the route and explain the request flow.

# Summary

Django's essential path is request -> URL resolver -> view -> response. Models and templates are optional steps. DRF reuses this foundation and specializes it for APIs.

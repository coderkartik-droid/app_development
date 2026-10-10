# Introduction

Django URL patterns map request paths to views. URL configuration (URLconf) is the routing table that lets a project organize pages and API endpoints.

# Why this concept exists

Separating paths from view code makes routes discoverable, testable, and maintainable. Named routes let code generate paths without duplicating strings.

# Real-world analogy

A URLconf is a building directory: a visitor follows the address to the correct office, then the office handles the request.

# Internal Working

```text
incoming path -> root urlpatterns -> included app urlpatterns -> matching view
```

Django tests patterns in order and selects a match. Captured path values become view arguments. Query strings are read from `request.GET`, not generally part of the route match.

# Syntax

```python
path("items/<int:item_id>/", views.item_detail, name="item-detail")
```

# Complete Example 1

Create a `catalog` app, register it, and connect these modules:

```python
# catalog/views.py
from django.http import HttpResponse


def item_detail(request, item_id):
    return HttpResponse(f"Item id: {item_id}")
```

```python
# catalog/urls.py
from django.urls import path
from . import views

app_name = "catalog"
urlpatterns = [
    path("items/<int:item_id>/", views.item_detail, name="item-detail"),
]
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("catalog/", include("catalog.urls"))]
```

**Line-by-line:** The view accepts both `request` and the captured integer `item_id`; it returns a response. The app URLconf imports `path`, declares a namespace, and maps an integer path segment to the view. Root URLconf mounts the app routes under `catalog/`. **Flow:** `/catalog/items/7/` -> root prefix -> typed capture `7` -> view -> response. **Internal:** `<int:...>` converter validates and converts the segment before invocation. **Professional tip:** Use named routes and namespaces; don't route API resources through ambiguous catch-all patterns.

# Complete Example 2

Read optional query-string filters:

```python
from django.http import JsonResponse


def search(request):
    term = request.GET.get("q", "").strip()
    return JsonResponse({"query": term})
```

Connect with `path("search/", search, name="search")`; a request to `/search/?q=books` returns the submitted term. **Flow:** route matches `/search/`; Django exposes query parameters via `request.GET`; view returns JSON. **Internal:** Query strings do not alter URL pattern matching. **Tip:** Validate length and allowed filters before using values in a query.

# Advanced Example

Reverse a namespaced URL with `reverse("catalog:item-detail", kwargs={"item_id": 7})`. Reversal uses route name and arguments to build a path. Prefer this over hard-coded strings in templates and redirects.

# Common Mistakes

- Forgetting trailing slash conventions or app URL inclusion.
- Confusing path parameters with query parameters.
- Duplicate route names without namespaces.
- Capturing a value but omitting it from the view signature.

# Best Practices

- Give routes stable descriptive names.
- Group app routes in app-level URLconfs.
- Use typed path converters and validate query parameters.
- Order overlapping routes intentionally.

# Interview Questions

1. What does URLconf do? Maps request paths to views.
2. How read a query parameter? `request.GET.get("key")`.
3. What does `include` do? Delegates part of routing to another URLconf.
4. Why name routes? Reverse paths without hard-coded URLs.

# Practice

Add `/api/books/<int:book_id>/` and `/api/books/?author=...`; identify path vs query values.

# Summary

URL patterns route paths, converters capture typed segments, includes compose route trees, and namespaced route names support reliable reversal.

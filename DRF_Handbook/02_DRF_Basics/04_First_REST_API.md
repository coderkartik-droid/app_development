# Introduction

A REST-style API exposes resources through URLs and uses HTTP methods to act on them. Your first DRF endpoint can be a simple read-only response; later chapters add serializers and database-backed resource operations.

# Why this concept exists

Clients such as web frontends, mobile apps, and command-line tools need a predictable contract for reading and changing server data. HTTP methods and status codes make that contract explicit.

# Real-world analogy

A library catalog offers a stable address for a book collection and clear actions for listing or retrieving entries. The API is the catalog desk shared by different client applications.

# Internal Working

```text
GET /api/books/ -> URLconf -> DRF view -> data source -> Response -> renderer -> JSON
```

DRF routes the request, checks the method, executes the handler, and finalizes the response. This first endpoint uses sample data so the HTTP/DRF path is visible without model setup.

# Request Lifecycle (where applicable)

The client sends `GET` and an `Accept` header. DRF matches the URL, constructs its request wrapper, dispatches to the GET handler, creates response data, and selects a renderer based on content negotiation.

# Syntax

```python
@api_view(["GET"])
def book_list(request):
    return Response({"results": []})
```

# Complete Example 1

Prerequisite: Django and DRF installed, `"rest_framework"` and `"books"` in `INSTALLED_APPS`, and an app named `books`.

```python
# books/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["GET"])
def book_list(request):
    books = [
        {"id": 1, "title": "Django Foundations"},
        {"id": 2, "title": "Practical APIs"},
    ]
    return Response({"count": len(books), "results": books})
```

```python
# books/urls.py
from django.urls import path
from .views import book_list

urlpatterns = [path("books/", book_list, name="book-list")]
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("books.urls"))]
```

Run `python manage.py runserver`; request `/api/books/`.

**Line-by-line:** Import DRF's decorator and response class; allow GET only; construct two primitive book mappings; wrap them in a response envelope with count and results. URL configuration maps a named resource route and mounts it under `/api/`. **Execution flow:** GET -> project URL include -> app route -> DRF dispatch -> list creation -> Response -> JSON renderer -> client. **Internal working:** No database is queried, so results reset with code changes. **Professional explanation:** This teaches endpoint shape only; use a model and serializer for real persisted records. Removing the decorator loses DRF method enforcement and request wrapping.

# Complete Example 2

Read one sample resource:

```python
@api_view(["GET"])
def book_detail(request, book_id):
    if book_id != 1:
        return Response({"detail": "Not found"}, status=404)
    return Response({"id": 1, "title": "Django Foundations"})
```

Route with `path("books/<int:book_id>/", book_detail)`. The converter supplies an integer; the view returns 404 for unknown IDs. **Flow:** path capture -> lookup decision -> response status/body. **Tip:** Use Django `get_object_or_404` or a queryset for model-backed data.

# Complete Example 3

Test the collection:

```python
from rest_framework.test import APITestCase
from django.urls import reverse


class BookListTests(APITestCase):
    def test_returns_collection(self):
        response = self.client.get(reverse("book-list"))
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.data["count"], 2)
```

The test client dispatches through Django and DRF routing; `response.data` is unrendered response data convenient for assertions. **Tip:** Also assert method restrictions and schema shape.

# Advanced Example

A resource endpoint should define collection and detail routes, allowed methods, permissions, validation, pagination, and error semantics. REST is a design style; DRF does not automatically make a URL design RESTful.

# Under the Hood

`@api_view` wraps the function as an API view. `Response` delays rendering; DRF's finalization applies renderer and response headers after handler return.

# Performance Notes

Sample lists are cheap; real collection endpoints should paginate and avoid N+1 related queries. Keep response payloads focused.

# Best Practices

- Use plural nouns for collections and stable identifiers for detail resources.
- Return accurate status codes and consistent response shapes.
- Use serializers for model conversion and validation.
- Add tests at the HTTP boundary.

# Common Mistakes

- Using a route that does not match the mounted URL prefix.
- Returning a raw model object/list without serialization.
- Treating hard-coded sample data as persistence.
- Returning 200 for missing resources.

# Debugging

Verify URL inclusion, route name, captured parameter type, allowed method, and `Accept` header. Inspect `response.status_code` and `response.data`.

# Interview Questions

1. What is a resource? A domain object or collection exposed through API routes.
2. What method commonly reads resources? GET.
3. What does REST require? Consistent resource-oriented constraints and HTTP semantics; it is not simply “JSON over HTTP.”
4. Why serializer? Validate and convert complex/model data to primitive representations.

# Practice Exercises

1. Add a `/api/books/<id>/` detail route.
2. Return 404 for unknown IDs.
3. Add an API test for list and detail.

# Chapter Summary

A first DRF API connects a named URL to a method-limited handler and returns a rendered `Response`. Begin with clear resource contracts, then introduce persistence, serializers, permissions, and pagination.

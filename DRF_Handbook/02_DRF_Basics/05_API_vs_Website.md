# Introduction

A traditional Django website commonly returns HTML for a browser to display. An API returns structured representations—often JSON—that another program can consume. The same Django project may provide both kinds of endpoints.

# Why this concept exists

Browsers, mobile apps, and other services need different response formats. Separating a user-facing page from a data contract lets each client present the data appropriately.

# Real-world analogy

A website is a prepared meal served at a table; an API is a standardized ingredient box that different kitchens can use in their own recipes.

# Internal Working

```text
Browser page: request -> Django view -> template -> HTML response
API client:   request -> DRF view -> serializer/data -> renderer -> JSON response
```

Both paths use Django middleware and URL routing. DRF adds API parsing, content negotiation, serializer integration, and API policy hooks.

# Request Lifecycle (where applicable)

An HTML browser request commonly uses cookies and form submissions, including CSRF protection for unsafe methods. An API client sends representations such as JSON and receives status codes and structured error data. Authentication and CSRF considerations depend on the selected authentication mechanism; they are not interchangeable.

# Syntax

```python
# HTML
return render(request, "books.html", {"books": books})

# DRF
return Response({"results": serialized_books})
```

# Complete Example 1

Create two views returning different representations of the same sample data:

```python
# catalog/views.py
from django.http import JsonResponse
from django.shortcuts import render
from rest_framework.decorators import api_view
from rest_framework.response import Response

BOOKS = [{"id": 1, "title": "Django Basics"}]


def website_book_list(request):
    return render(request, "catalog/books.html", {"books": BOOKS})


@api_view(["GET"])
def api_book_list(request):
    return Response({"results": BOOKS})
```

```html
<!-- catalog/templates/catalog/books.html -->
<h1>Books</h1>
{% for book in books %}<p>{{ book.title }}</p>{% endfor %}
```

```python
# catalog/urls.py
from django.urls import path
from .views import api_book_list, website_book_list

urlpatterns = [
    path("books/", website_book_list, name="website-books"),
    path("api/books/", api_book_list, name="api-books"),
]
```

**Line-by-line:** `BOOKS` is sample data. The first view passes it to an HTML template; the second uses DRF's method wrapper and `Response`. URL patterns expose distinct routes. **Execution flow:** both requests go through Django routing; the website path renders HTML, while the API path finalizes JSON. **Internal working:** A DRF `Response` is rendered after content negotiation; `render` resolves a template immediately. **Professional tip:** Share domain/query logic through a service/repository rather than duplicate it as formats diverge. Removing `@api_view` removes API dispatch behavior from the API function.

# Complete Example 2

An API response can provide a stable machine-readable error:

```python
return Response({"detail": "Book not found"}, status=404)
```

A web page might render an HTML not-found template instead. Both communicate the same outcome but have different representation contracts. **Tip:** Clients should use status codes and documented fields rather than scraping human-readable HTML.

# Complete Example 3

Use a browser request to a browsable API endpoint during development. DRF can render an HTML interface when the browser accepts HTML, while a client requesting JSON receives a JSON representation. This is content negotiation, not a separate resource contract.

# Advanced Example

Cookie-based browser authentication and token-based API clients have different CSRF and credential-storage properties. Choose an authentication scheme with a threat model; do not disable CSRF globally just to make a browser request work.

# Under the Hood

The core difference is representation and client contract, not that one has a database and the other does not. Websites and APIs can both use models, validation, permissions, and the same business rules.

# Performance Notes

API payloads should avoid unnecessary fields. HTML rendering can be server-side; API response costs include serialization and network transfer.

# Best Practices

- Design separate, explicit response contracts.
- Reuse domain logic while keeping representation layers distinct.
- Apply appropriate authentication and CSRF protections.
- Do not expose internal model fields by default.

# Common Mistakes

- Assuming every endpoint returning JSON is automatically RESTful or secure.
- Treating HTML and API authentication identically.
- Returning template markup to a machine client.
- Duplicating business rules in website and API views.

# Debugging

Inspect `Accept`, `Content-Type`, status, cookies/authorization, and response body. Confirm which URL pattern and view handled the request.

# Interview Questions

1. Website vs API response? Usually HTML for people vs structured representation for programs.
2. Can both use Django ORM? Yes.
3. What does content negotiation do? Selects a response representation based on request preferences and available renderers.
4. Does JSON imply security? No.

# Practice Exercises

1. Return one sample item as HTML and JSON.
2. Compare content type and payload shape.
3. Explain CSRF concerns for cookie-authenticated unsafe requests.

# Chapter Summary

Websites and APIs share Django's foundation but serve different client contracts. Keep presentation distinct from machine representations and choose security behavior based on the authentication mechanism.

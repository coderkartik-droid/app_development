# Introduction

DRF's `Request` wraps Django's `HttpRequest` and adds API-friendly access to parsed body data, authentication, and content negotiation information. It still provides access to underlying Django request properties.

# Why this concept exists

APIs can receive JSON, form data, and other media types. A unified `request.data` interface avoids endpoint code manually choosing a parser and decoding each format.

# Real-world analogy

The request object is an intake folder with the original envelope plus labeled, parsed contents and caller credentials.

# Internal Working

```text
Django HttpRequest -> DRF Request
  .method .path .headers
  .query_params -> query string
  .data          -> parser-decoded body (lazy)
  .user/.auth    -> authentication result
```

DRF's wrapper delegates many properties to Django's request. Body parsing usually occurs when `request.data` is first accessed.

# Request Lifecycle (where applicable)

APIView initializes its DRF request, selects authenticators, and applies permissions. The handler reads `request.data` or `request.query_params`; a parser handles body bytes and an authenticator may populate user/auth data.

# Syntax

```python
name = request.data.get("name")
search = request.query_params.get("search", "")
```

# Complete Example 1

Accept a query parameter and return a normalized value:

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["GET"])
def search_items(request):
    term = request.query_params.get("q", "").strip()
    return Response({
        "query": term,
        "method": request.method,
        "path": request.path,
    })
```

```python
# api/urls.py
from django.urls import path
from .views import search_items

urlpatterns = [path("items/search/", search_items, name="search-items")]
```

Request `/api/items/search/?q=pen` after mounting app URLs under `/api/`.

**Line-by-line:** The endpoint permits GET. `query_params` accesses URL query values and defaults missing `q` to an empty string. `strip` normalizes whitespace. The response includes query, method, and path. **Flow:** URL route matches path -> request wrapper exposes query string -> handler builds mapping -> renderer emits JSON. **Internal working:** Query parameters are separate from parsed request body. **Professional tip:** Validate query length and supported filters; never assume a client supplied a scalar in the expected shape. Removing the default can produce `None` and break string operations.

# Complete Example 2

Read JSON body data:

```python
@api_view(["POST"])
def create_label(request):
    label = request.data.get("label")
    if not isinstance(label, str):
        return Response({"label": ["Must be text."]}, status=400)
    return Response({"label": label.strip()}, status=201)
```

`request.data` invokes parser selection from the request media type. Type validation is necessary before calling string methods. **Flow:** bytes -> parser -> Python object -> validation -> response. **Tip:** Replace manual checks with serializers as fields and rules grow.

# Complete Example 3

Read request headers:

```python
client_version = request.headers.get("X-Client-Version", "unknown")
```

Headers provide transport metadata. Treat custom header values as untrusted input; do not use a client-supplied role or user ID as authorization proof.

# Advanced Example

Authentication classes populate `request.user` and `request.auth` according to policy. A missing/invalid credential can produce an unauthenticated user or an exception depending on the configured scheme. Permissions should inspect authenticated identity and resource ownership.

# Under the Hood

`Request` wraps `HttpRequest`, carries parser/authenticator context, and exposes lazy body parsing. `.query_params` is an alias emphasizing query-string semantics over Django's `.GET`.

# Performance Notes

Access body data only when needed; large payload parsing costs memory/time. Bound query parameters and use pagination for result sets.

# Best Practices

- Use `.data` for request bodies and `.query_params` for URL filters.
- Validate types, ranges, and lengths.
- Treat headers and body fields as untrusted.
- Use authentication/permission classes for identity and access control.

# Common Mistakes

- Reading JSON from `request.POST`.
- Confusing query parameters with body fields.
- Assuming every request is authenticated.
- Accessing a value without checking its type/absence.

# Debugging

Inspect method, path, query string, headers, content type, raw body safely, parser configuration, and authentication result. Avoid logging credentials or personal data.

# Interview Questions

1. What is `request.data`? Parsed body data.
2. What is `request.query_params`? Query-string mapping.
3. When does parsing occur? Usually lazily on body-data access.
4. Who sets `request.user`? Configured authentication classes through DRF's authentication process.

# Practice Exercises

1. Accept `?page=2` and validate a positive integer.
2. Accept a JSON POST and reject wrong field types.
3. Read a custom header while treating it as untrusted.

# Chapter Summary

DRF's `Request` unifies parsed body, query parameters, headers, and authentication context. Know which source a value came from, validate it, and avoid treating client input as trusted.

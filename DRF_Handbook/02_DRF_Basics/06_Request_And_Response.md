# Introduction

DRF's `Request` wraps Django's `HttpRequest` with API-friendly parsed data and authentication information. DRF's `Response` holds unrendered Python data that is converted into the negotiated representation after the view returns.

# Why this concept exists

API views need to consume different request formats and return consistent responses without manually decoding bytes or selecting encoders in every endpoint.

# Real-world analogy

`Request` is an unpacked, labeled delivery arriving at a desk; `Response` is a prepared package that is wrapped in the format the recipient requested.

# Internal Working

```text
HttpRequest -> DRF Request -> handler
                           -> Response(data, status)
                           -> content negotiation -> renderer -> HttpResponse
```

Request body parsing is generally lazy: accessing `request.data` invokes a parser based on the content type. Rendering is delayed until DRF finalizes the response.

# Request Lifecycle (where applicable)

DRF dispatch initializes request context, performs authentication and permissions, invokes the handler, then finalizes the response. Exceptions are mapped through DRF's exception handler when recognized.

# Syntax

```python
payload = request.data
return Response({"received": payload}, status=200)
```

# Complete Example 1

This endpoint accepts a JSON field and returns a validated simple response. Use a project with DRF installed and registered.

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["POST"])
def echo_title(request):
    title = request.data.get("title")
    if not isinstance(title, str) or not title.strip():
        return Response(
            {"title": ["A non-empty string is required."]},
            status=400,
        )
    return Response({"title": title.strip()}, status=201)
```

```python
# api/urls.py
from django.urls import path
from .views import echo_title

urlpatterns = [path("titles/", echo_title, name="echo-title")]
```

Send `Content-Type: application/json` with `{"title":"Example"}`.

**Line-by-line:** The decorator allows POST only. `request.data` asks DRF to parse the body according to `Content-Type`. The handler checks the Python type and blank value before use. Invalid data gets a field-shaped error and 400; valid data is trimmed and returned with 201. **Flow:** body bytes -> JSON parser -> `Request.data` -> validation -> `Response` -> renderer -> JSON bytes. **Internal working:** Parsing may occur on first data access, and parser errors are raised as API exceptions. **Professional tip:** Use serializers for non-trivial validation and persistence. Removing the type check can cause runtime errors on unexpected payloads.

# Complete Example 2

Return an explicit no-content response:

```python
from rest_framework import status
from rest_framework.response import Response


def delete_result():
    return Response(status=status.HTTP_204_NO_CONTENT)
```

The response signals successful deletion without a representation. In a real endpoint, use a method-decorated/API view and perform the deletion first. **Tip:** Do not include a body with 204; clients expect no content.

# Complete Example 3

Use status constants:

```python
return Response({"detail": "Not found"}, status=404)
```

Prefer `status.HTTP_404_NOT_FOUND` for readability and fewer magic numbers. The response body and status communicate different parts of the contract.

# Advanced Example

Do not return a Django model instance directly. Convert it to primitive values with a serializer or explicit mapping; renderer supports primitive structures, not arbitrary ORM semantics.

# Under the Hood

`Response` subclasses Django response behavior but stores data until renderer selection. `Request` delegates access to underlying request details while exposing parsed `data`, query parameters, and DRF authentication properties.

# Performance Notes

Avoid parsing the same large body repeatedly and avoid returning oversized representations. Use streaming/file responses only when the endpoint's content and renderer design require them.

# Best Practices

- Read submitted payloads through `request.data`.
- Read query strings through `request.query_params`.
- Validate before side effects.
- Return meaningful status and structured errors.

# Common Mistakes

- Using `request.POST` for JSON bodies.
- Returning raw model objects.
- Parsing body manually without need.
- Returning success status before a write actually succeeds.

# Debugging

Check request `Content-Type`, parser configuration, body shape, handler method, `Response.status_code`, and negotiated renderer. Distinguish parser errors from validation errors.

# Interview Questions

1. What is `request.data`? Parsed request body data in DRF.
2. Why is parsing lazy? It can defer cost until data is needed.
3. Why use `Response`? It supports DRF renderer negotiation and API finalization.
4. Can response data be any Python object? It should be renderable primitive data, commonly supplied via a serializer.

# Practice Exercises

1. Accept a POST JSON title and reject non-string/blank values.
2. Return 201 on accepted creation and 400 on invalid input.
3. Test malformed JSON and wrong content type.

# Chapter Summary

DRF wraps Django requests, parses body data on demand, and delays response rendering until content negotiation. Validate input, use `Response` with primitive/serialized data, and select accurate status codes.

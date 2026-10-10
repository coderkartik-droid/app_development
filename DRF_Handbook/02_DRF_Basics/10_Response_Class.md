# Introduction

DRF's `Response` class holds unrendered API data and an HTTP status. Unlike immediately rendered plain text, its content is rendered later using the format selected during content negotiation.

# Why this concept exists

An endpoint should return Python data without manually encoding JSON or guessing the client's preferred representation. A delayed response lets DRF apply renderers, headers, and API behavior consistently.

# Real-world analogy

`Response` is a parcel before packaging: the contents are prepared, and the delivery system wraps them in the requested format.

# Internal Working

```text
Response(data, status, headers)
 -> APIView.finalize_response()
 -> selected renderer.render(data)
 -> rendered bytes + Content-Type + HTTP status
```

# Request Lifecycle (where applicable)

The view returns a DRF `Response`. During finalization, DRF attaches renderer and media-type context chosen for the request, then the response is rendered for the client.

# Syntax

```python
return Response({"ok": True}, status=200)
```

# Complete Example 1

Create a status endpoint returning a JSON-compatible mapping:

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework import status


@api_view(["GET"])
def readiness(request):
    payload = {"ready": True, "component": "catalog"}
    return Response(payload, status=status.HTTP_200_OK)
```

```python
# api/urls.py
from django.urls import path
from .views import readiness

urlpatterns = [path("ready/", readiness, name="readiness")]
```

**Line-by-line:** Import the decorator, response, and status constants. Allow GET only. Construct a primitive mapping. Return it with explicit 200. URL routing exposes the endpoint. **Flow:** GET -> decorator dispatch -> handler -> `Response.data` -> renderer -> HTTP response. **Internal working:** The response does not need to know JSON in the view; renderer selection occurs later. **Professional tip:** Avoid explicitly passing 200 when it is the default unless it improves clarity; always make non-default codes explicit. Removing the DRF decorator means finalization does not happen as expected for this response type.

# Complete Example 2

Return validation details:

```python
return Response(
    {"email": ["Enter a valid email address."]},
    status=status.HTTP_400_BAD_REQUEST,
)
```

The mapping groups errors by field; the status communicates invalid input. In production, prefer serializer validation so error format and field rules remain consistent.

# Complete Example 3

Return no body after deletion:

```python
return Response(status=status.HTTP_204_NO_CONTENT)
```

A 204 response has no representation. Do not pass a success dictionary with this status; use 200 if a body is required.

# Advanced Example

`Response` accepts status, headers, and content type options. Usually allow content negotiation to choose the media type; forcing a format on one endpoint can surprise clients and interfere with renderer policy.

# Under the Hood

`Response` is a specialized Django `SimpleTemplateResponse`-style response that stores data before rendering. DRF's `APIView` finalization attaches accepted renderer/context; rendering turns data into a byte body.

# Performance Notes

Large nested structures incur serialization and memory cost. Return bounded, focused payloads and use pagination.

# Best Practices

- Pass primitives or serializer output, not model instances.
- Use DRF status constants.
- Keep errors structured and stable.
- Let configured renderers negotiate formats unless there is a clear requirement.

# Common Mistakes

- Returning a dict directly from a Django view.
- Returning a model/queryset without serialization.
- Using 204 with a body.
- Confusing `Response.data` with already encoded JSON bytes.

# Debugging

Inspect `response.data` in tests, rendered response content, status, headers, and `Content-Type`. If rendering fails, inspect values that are not primitive or serializable.

# Interview Questions

1. What is delayed rendering? Data is stored until DRF selects and runs a renderer.
2. What should `Response.data` contain? Primitive representations, often serializer output.
3. What makes `Response` different from `JsonResponse`? DRF renderer negotiation and finalization integration.
4. Can a response have headers? Yes; use supported response header options.

# Practice Exercises

1. Return 200, 201, 400, and 204 examples.
2. Verify status, body, and content type with API tests.
3. Return an unserializable object and inspect the rendering error; then fix it with primitive data.

# Chapter Summary

DRF `Response` stores API data until a renderer converts it. Return serializable primitives, use accurate status codes, and let DRF finalize the response through its request-aware pipeline.

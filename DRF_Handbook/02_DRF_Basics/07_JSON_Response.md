# Introduction

JSON is a text format for objects, arrays, strings, numbers, booleans, and null. DRF commonly returns JSON by placing Python primitive data in a `Response`; a renderer serializes it.

# Why this concept exists

Clients need a portable representation that can be decoded across programming languages. JSON provides a widely supported format with a simple data model.

# Real-world analogy

JSON is a standardized shipping label: different systems can read the same structured fields even if they were built with different tools.

# Internal Working

```text
Python dict/list -> DRF Response.data -> JSONRenderer -> UTF-8 response body
```

DRF's `JSONRenderer` converts supported primitives to JSON and sets an appropriate media type. Database model instances must first be converted to primitives, typically by a serializer.

# Request Lifecycle (where applicable)

The endpoint returns `Response(data)`. DRF selects a renderer based on request negotiation, calls it to produce bytes, and emits the HTTP response with status and headers.

# Syntax

```python
return Response({"id": 3, "name": "Pen"})
```

# Complete Example 1

Create a JSON endpoint with nested primitive data:

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(["GET"])
def product_summary(request):
    data = {
        "id": 3,
        "name": "Pen",
        "available": True,
        "tags": ["stationery", "writing"],
        "price": None,
    }
    return Response(data)
```

```python
# api/urls.py
from django.urls import path
from .views import product_summary

urlpatterns = [path("product/", product_summary, name="product-summary")]
```

**Line-by-line:** Import DRF view and response helpers; restrict the endpoint to GET; construct a mapping of JSON-compatible values; return it. The URL provides a stable route name. **Flow:** request -> view -> mapping -> JSON renderer -> response body. **Internal working:** Python `None` becomes JSON `null`; booleans and lists retain their JSON equivalents. **Professional tip:** API response keys and nullability are public contracts—document and test them. Removing `Response` means Django does not know how to return the mapping directly.

# Complete Example 2

Return a list envelope:

```python
return Response({
    "count": 2,
    "results": [
        {"id": 1, "name": "Pen"},
        {"id": 2, "name": "Notebook"},
    ],
})
```

An envelope can later carry pagination links or metadata. For large resources, use DRF pagination instead of hard-coding a full collection. **Flow:** list data -> JSON array inside object -> renderer. **Tip:** Keep field types stable; don't sometimes return a number and sometimes a formatted string.

# Complete Example 3

Inspect the test representation:

```python
response = self.client.get("/api/product/")
self.assertEqual(response.json()["available"], True)
```

`response.json()` decodes a JSON response in Django's test client. Test content type and status too; parsing JSON alone does not prove success.

# Advanced Example

Use a serializer for model output so field selection, nested representations, and type conversion are explicit. Avoid relying on the JSON renderer to serialize arbitrary Python objects such as querysets or model instances.

# Under the Hood

JSON has no native date, decimal, UUID, or binary types. DRF serializers convert such values into a chosen primitive form; the renderer then encodes those primitives as JSON text.

# Performance Notes

Large payloads increase serialization time and network transfer. Paginate and select only fields clients need; avoid deeply nested object graphs by default.

# Best Practices

- Return dictionaries/lists and primitive values through `Response`.
- Use serializers for structured/domain data.
- Specify stable field names, types, and null behavior.
- Test actual JSON and status contracts.

# Common Mistakes

- Returning Python `None` unexpectedly where clients expect a field.
- Returning `Decimal` or model objects without an explicit representation policy.
- Using inconsistent keys or types across responses.
- Omitting pagination for large collections.

# Debugging

Inspect `Content-Type`, response body, status, and renderer settings. Use test client JSON decoding and verify nested field types.

# Interview Questions

1. What is JSON? A text representation of structured primitive data.
2. What does DRF JSONRenderer do? Renders supported response data into JSON bytes.
3. How represent null in Python? `None`, rendered as JSON `null`.
4. Why use a serializer? To explicitly convert and validate structured data.

# Practice Exercises

1. Return a product with string, integer, boolean, list, and nullable fields.
2. Write tests for field types and content type.
3. Add an envelope with results and count.

# Chapter Summary

DRF turns primitive `Response.data` into JSON through a renderer. Define stable field contracts, serialize models explicitly, and keep large responses bounded.

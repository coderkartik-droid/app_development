# Introduction

Response formatting defines the shape, status, headers, and media type returned by an endpoint. A stable format makes clients predictable and errors diagnosable.

# Why this concept exists

Different clients need a consistent contract for success, pagination, validation errors, and empty results.

# Real-world analogy

Response formatting is a standard receipt: every cashier gives the same fields in the same places, so accounting software can read it.

# Internal Working

Views return DRF `Response` objects containing Python primitives or serializer data. Content negotiation chooses a renderer; finalization sets renderer context and converts the representation to bytes.

# Request Lifecycle

View result -> Response(data, status, headers) -> content negotiation renderer -> HTTP body/headers -> client.

# Syntax

```python
return Response(serializer.data, status=status.HTTP_200_OK)
```

# Complete Example 1

```python
from rest_framework import generics
from rest_framework.response import Response
from rest_framework import status
from .models import Product
from .serializers import ProductSerializer

class ProductDetailView(generics.RetrieveAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer

    def retrieve(self, request, *args, **kwargs):
        instance = self.get_object()
        data = self.get_serializer(instance).data
        return Response(data, status=status.HTTP_200_OK)
```

The view retrieves through its scoped object helper and serializes the instance. Explicit `200` is conventional though the default is already 200. Returning `Response` lets DRF render according to negotiated media type; returning a raw Python dict from a Django view would not provide identical behavior.

# Complete Example 2

```python
from rest_framework.response import Response
from rest_framework import status

def create_result(serializer):
    serializer.is_valid(raise_exception=True)
    instance = serializer.save()
    return Response(serializer.data, status=status.HTTP_201_CREATED,
                    headers={"Location": f"/api/products/{instance.pk}/"})
```

This helper demonstrates `201 Created` and a `Location` header. It assumes a serializer with model instance and primary key; a view may use `reverse()` for absolute/route-safe URL construction. Validation errors are raised for DRF's exception handler.

# Complete Example 3

```python
class ProductDeleteView(generics.DestroyAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
```

The built-in destroy response is normally `204 No Content`; it has no JSON object body. Clients should not attempt to parse a missing body as a resource. This code is complete in an app with imports/models configured.

# Advanced Example

Use one documented error envelope and customize `EXCEPTION_HANDLER` only when all endpoints can preserve a consistent contract. Include machine-readable codes; avoid returning stack traces or raw database messages.

# Under the Hood

DRF delays rendering until response finalization, when renderer and media type are known. Serializer `.data` is a primitive representation, while a renderer encodes it (JSONRenderer to JSON bytes).

# Performance Notes

Avoid redundant serialization, huge nested representations, and expensive renderer formats in high-volume APIs. Keep response fields minimal for the use case.

# Best Practices

- Use standard status codes and consistent response envelopes.
- Return serializer representations rather than model objects.
- Define pagination and error shapes in API documentation.
- Keep sensitive/internal data out of responses.

# Common Mistakes

- Returning 200 for every outcome.
- Returning 204 with a body clients expect to parse.
- Manually JSON-encoding data then wrapping it in `Response`.

# Debugging

Inspect `Accept`, negotiated renderer, response status/headers, serializer `.data`, and rendered body. Verify error handlers do not mask useful validation details.

# Interview Questions

1. What does DRF Response contain before rendering? Python primitive data and response metadata.
2. When is JSON encoding performed? During response rendering.
3. What is appropriate for successful creation? Usually 201 Created.

# Practice Exercises

1. Define success/error examples for a CRUD resource.
2. Add a Location header using URL reversing.
3. Test 204 has an empty body.

# Chapter Summary

Response formatting is a public API contract. Return clear status codes, stable primitive shapes, appropriate headers, and let DRF render the negotiated representation.

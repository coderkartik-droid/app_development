# Introduction

Custom responses shape status codes, headers, and body data for an API operation while preserving a consistent client contract.

# Why this concept exists

Clients need meaningful outcomes for creation, deletion, validation, asynchronous work, and caching.

# Real-world analogy

A delivery service gives different receipts for “accepted,” “completed,” and “not deliverable,” even though each is a response to one request.

# Internal Working

DRF `Response` stores primitive data and metadata. During finalization, content negotiation chooses a renderer and converts data to the response body.

# Authentication Flow (where applicable)

Client -> authentication -> permission -> view/domain operation -> Response(data,status,headers) -> renderer -> client.

# Syntax

```python
return Response(data, status=status.HTTP_201_CREATED, headers=headers)
```

# Complete Example 1

```python
from rest_framework import generics, status
from rest_framework.response import Response
from django.urls import reverse

class ProductCreateView(generics.CreateAPIView):
    serializer_class = ProductSerializer

    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        product = serializer.save()
        location = reverse("product-detail", kwargs={"pk": product.pk}, request=request)
        return Response(
            self.get_serializer(product).data,
            status=status.HTTP_201_CREATED,
            headers={"Location": location},
        )
```

The serializer validates request data, save persists the product, and `reverse` builds a route-safe Location. This assumes a matching named detail URL and serializer. Do not return 201 until persistence succeeds; prefer framework behavior unless customization is needed.

# Complete Example 2

```python
from rest_framework.response import Response
from rest_framework import status

def accepted_job(job_id):
    return Response(
        {"job_id": job_id, "state": "queued"},
        status=status.HTTP_202_ACCEPTED,
        headers={"Retry-After": "3"},
    )
```

202 means processing is accepted but not complete. The body gives a tracking identifier and header suggests when to poll. The API should provide a status endpoint and document eventual failure semantics.

# Complete Example 3

```python
def delete_response():
    return Response(status=status.HTTP_204_NO_CONTENT)
```

204 communicates successful completion with no representation. Do not attach a JSON body to a 204 response; clients may ignore it or reject it.

# Advanced Example

Standardize error response codes and request correlation headers. Avoid inventing envelopes inconsistently across endpoints; version changes if a public response contract must break.

# Under the Hood

The response is rendered after the view returns, so `Response.data` and final bytes are distinct. Renderer selection follows content negotiation and can vary with `Accept`.

# Performance Notes

Keep bodies small, avoid serializing the same instance repeatedly, and use streaming responses only for suitable large content with clear resource lifecycle.

# Security Notes

Do not expose stack traces, secrets, or internal identifiers. Ensure caching headers do not make private responses publicly cacheable.

# Best Practices

- Use semantically correct status codes.
- Include headers only when documented and useful.
- Return serializer representations, not model objects.
- Keep success and error envelopes consistent.

# Common Mistakes

- Returning 200 for every result.
- Returning a body with 204.
- Manually JSON-encoding data before wrapping it in `Response`.

# Debugging

Inspect `Response.data`, status, headers, negotiated renderer, and actual HTTP body.

# Interview Questions

1. What is returned for asynchronous accepted work? Commonly 202.
2. What does Location identify after creation?
3. When is Response data rendered?

# Practice Exercises

1. Add Location to a creation response.
2. Design a 202 job response and status route.
3. Test 204 body is empty.

# Chapter Summary

Responses are API contracts, not arbitrary dictionaries. Use meaningful status and headers, let DRF render primitive data, and keep behavior consistent and secure.

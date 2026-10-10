# Introduction

Path parameters identify the resource or nested location addressed by a URL, for example the `42` in `/products/42/`.

# Why this concept exists

Stable identifiers make resource URLs addressable, bookmarkable, and easy for clients to compose.

# Real-world analogy

A path parameter is a house number in a street address: it identifies the destination, while query parameters are delivery preferences.

# Internal Working

Django's URL resolver matches converters such as `<int:pk>` and passes captured values as keyword arguments to the view. DRF's generic views use `lookup_field` and `lookup_url_kwarg` when retrieving an object.

# Request Lifecycle

`/products/42/` -> URL converter parses `42` -> detail view receives `pk=42` -> scoped queryset lookup -> permission check -> serializer -> response.

# Syntax

```python
path("products/<int:pk>/", ProductDetailView.as_view())
```

# Complete Example 1

```python
from django.urls import path
from rest_framework.generics import RetrieveAPIView
from .models import Product
from .serializers import ProductSerializer

class ProductDetailView(RetrieveAPIView):
    queryset = Product.objects.filter(active=True)
    serializer_class = ProductSerializer

urlpatterns = [path("products/<int:pk>/", ProductDetailView.as_view(), name="product-detail")]
```

The integer converter rejects non-integer path segments before view execution. `RetrieveAPIView` uses the captured `pk` to find an active product and return 404 otherwise. The queryset can also enforce tenant visibility. Omitting the URL converter or using an unrestricted custom lookup can create ambiguous or unsafe access.

# Complete Example 2

```python
class ProductBySkuView(RetrieveAPIView):
    queryset = Product.objects.filter(active=True)
    serializer_class = ProductSerializer
    lookup_field = "sku"
    lookup_url_kwarg = "sku"
```

Configure its route as `path("products/by-sku/<slug:sku>/", ProductBySkuView.as_view())`. `lookup_field` names the model field; `lookup_url_kwarg` names the captured URL variable. A unique database constraint on SKU avoids ambiguous matches.

# Complete Example 3

```python
from rest_framework.test import APIClient

response = APIClient().get("/api/products/not-an-integer/")
assert response.status_code == 404
```

With an integer converter this URL does not match the detail route, so Django typically returns 404 before DRF. Test valid, missing, malformed, and unauthorized identifiers separately; exact error rendering may be customized.

# Advanced Example

For nested routes, validate parent-child membership in the queryset or lookup logic. A child ID must not be retrievable merely because its parent appears in the path.

# Under the Hood

URL converters perform syntactic conversion, not authorization. Generic `get_object()` filters the queryset using lookup configuration, raises `Http404` on no match, and runs object-level permission checks.

# Performance Notes

Index lookup fields, prefer unique identifiers, and avoid multi-query parent/child checks when a scoped join can enforce membership.

# Best Practices

- Use converters matching identifier shape.
- Keep identifiers stable and non-sensitive.
- Enforce uniqueness and parent ownership in the database/queryset.
- Return consistent not-found behavior.

# Common Mistakes

- Believing an unguessable ID replaces authorization.
- Confusing path parameters with query parameters.
- Allowing a nested child lookup outside its specified parent.

# Debugging

Check URL pattern order, converter type, kwargs, `lookup_field`, queryset scope, and object permissions.

# Interview Questions

1. How does a path parameter differ from a query parameter? It identifies path/resource; query modifies request options.
2. What does `lookup_url_kwarg` control? The URL keyword used for lookup.
3. Does UUID secrecy secure a resource? No; permission checks are still needed.

# Practice Exercises

1. Change product lookup from integer ID to UUID.
2. Add a nested category/product route with parent scoping.
3. Test malformed and cross-tenant IDs.

# Chapter Summary

Path parameters locate a resource but do not grant access. URL converters parse shape; scoped querysets and permissions enforce visibility.

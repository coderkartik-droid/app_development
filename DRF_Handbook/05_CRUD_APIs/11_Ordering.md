# Introduction

Ordering determines the sequence in which collection results are returned. A client may request price ascending, newest first, or another documented sort.

# Why this concept exists

Databases do not promise a useful order unless one is specified. Predictable ordering supports usable interfaces and stable pagination.

# Real-world analogy

Ordering is choosing whether a shop catalogue is arranged by price, name, or newest arrival.

# Internal Working

DRF's `OrderingFilter` reads an ordering expression, checks fields against an allowlist, and applies `QuerySet.order_by()` before pagination.

# Request Lifecycle

Query string -> validate requested ordering fields -> SQL ORDER BY -> page slice -> serialization.

# Syntax

```http
GET /api/products/?ordering=-created_at,name
```

# Complete Example 1

```python
from rest_framework import filters, generics
from .models import Product
from .serializers import ProductSerializer

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [filters.OrderingFilter]
    ordering_fields = ["name", "price", "created_at"]
    ordering = ["name", "id"]
```

Only the listed fields can be requested; a leading `-` means descending. The default includes `id` as a tie-breaker for stable order. Add a real `created_at` model field before using it. Exposing arbitrary fields can leak details and create costly queries.

# Complete Example 2

```python
response = APIClient().get("/api/products/?ordering=-price,name")
assert response.status_code == 200
prices = [item["price"] for item in response.data]
assert prices == sorted(prices, reverse=True)
```

This demonstrates descending price ordering. If pagination is enabled, `response.data` may contain a `results` list; test the page appropriately. Equal-price records may have unspecified relative order unless a tie-breaker is included.

# Complete Example 3

```python
class ProductListView(generics.ListAPIView):
    queryset = Product.objects.order_by("name", "pk")
    serializer_class = ProductSerializer
```

This version fixes ordering server-side, which is useful when clients must not choose expensive sorts. A deterministic secondary key (`pk`) avoids unstable order among equal names.

# Advanced Example

For cursor pagination, ordering must be consistent and generally based on an immutable, unique or nearly unique field. Changing an item's sort value during traversal can affect which page a client observes.

# Under the Hood

OrderingFilter translates safe field names into ORM ordering. The database performs sort work, possibly using an index or a sort operation; ordering is applied before slicing.

# Performance Notes

Sort on indexed, selective fields where appropriate; avoid unbounded large-text sorts. Pair ordering with pagination and inspect query plans.

# Best Practices

- Explicitly allowlist fields.
- Set deterministic defaults and tie-breakers.
- Document ascending/descending notation.
- Avoid mutable ordering keys for cursor-based traversal.

# Common Mistakes

- Assuming default row order is stable.
- Letting users sort by every model field.
- Paginating without a deterministic order.

# Debugging

Inspect the URL expression, `ordering_fields`, generated `ORDER BY`, index use, and behavior for ties.

# Interview Questions

1. How is descending ordering requested? Prefix the field with `-`.
2. Why is stable ordering important for pagination? It prevents records shifting unpredictably between pages.
3. Who performs database sorting? The database via SQL `ORDER BY`.

# Practice Exercises

1. Add safe ordering by price and name.
2. Add a stable tie-breaker and test equal values.
3. Explain why sorting by an unindexed large field may be slow.

# Chapter Summary

Ordering is part of the API contract. Whitelist fields, choose deterministic defaults, and coordinate ordering with pagination and database indexes.

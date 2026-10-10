# Introduction

`GET` retrieves a representation without intentionally changing server state. Collection GET returns many resources; detail GET returns one resource. Both should be safe and repeatable from the client's perspective.

# Why this concept exists

Clients need to display current server data. A consistent GET contract supports web/mobile screens, caching, pagination, and integration without exposing database implementation details.

# Real-world analogy

GET is like reading a product label or asking a librarian for a catalogue entry: the request should not edit or remove the item.

# Internal Working

DRF resolves the collection/detail URL, checks policy, builds a queryset, evaluates it lazily when serialized, converts model fields into primitives, then renders JSON (or another negotiated format). Queryset scoping must happen before lookup to prevent object disclosure.

# Request Lifecycle

`GET /products/` -> list action -> scoped queryset -> optional filters/order/pagination -> serializer -> 200 response. `GET /products/42/` -> detail action -> scoped lookup -> serializer -> 200, or 404 when absent/invisible.

# Syntax

```http
GET /api/products/?active=true
Accept: application/json
Authorization: Bearer <token>
```

# Complete Example 1

```python
# serializers.py
from rest_framework import serializers
from .models import Product

class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ["id", "name", "sku", "price", "active"]
```

```python
# views.py, urls.py
from rest_framework import generics
from rest_framework.permissions import IsAuthenticatedOrReadOnly
from django.urls import path
from .models import Product
from .serializers import ProductSerializer

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.filter(active=True).order_by("name")
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]

urlpatterns = [path("products/", ProductListView.as_view(), name="product-list")]
```

`ListAPIView` implements GET list behavior. The queryset excludes inactive records and orders consistently; the serializer defines public output. `IsAuthenticatedOrReadOnly` allows anonymous reads, not writes (this view has no write handler anyway). Without `.order_by`, database ordering can vary. Add pagination before exposing a growing catalogue.

# Complete Example 2

```python
from rest_framework import generics
from django.urls import path
from .models import Product
from .serializers import ProductSerializer

class ProductDetailView(generics.RetrieveAPIView):
    queryset = Product.objects.filter(active=True)
    serializer_class = ProductSerializer

urlpatterns = [path("products/<int:pk>/", ProductDetailView.as_view(), name="product-detail")]
```

`RetrieveAPIView` maps GET to one object and returns 404 if the lookup fails. The `pk` converter limits the path segment to an integer. The active filter also hides inactive products as not found. Add explicit permission classes if products are private.

# Complete Example 3

```python
from rest_framework.test import APIClient

client = APIClient()
response = client.get("/api/products/42/")
assert response.status_code in (200, 404)
if response.status_code == 200:
    assert {"id", "name", "sku", "price", "active"} <= response.data.keys()
```

This test exercises URL resolution, view lookup, serialization, and response shape. A robust test should create the fixture and assert the expected status, not accept either value. Use `force_authenticate` when verifying authenticated policies.

# Advanced Example

Use `select_related("category")` for a single-valued foreign key or `prefetch_related("tags")` for a many-valued relation accessed by the serializer. Confirm query counts with representative data.

# Under the Hood

QuerySets are lazy: constructing one does not necessarily query the database. Serialization iterates the queryset, triggering SQL. DRF's pagination may slice/count it, and renderers transform primitive response data to bytes.

# Performance Notes

Paginate, select only needed columns when safe, add measured indexes, and prevent N+1 access from nested fields or serializer methods.

# Best Practices

- Keep GET side-effect free.
- Use stable ordering and pagination for collections.
- Return 404 for unavailable detail objects where appropriate.
- Do not serialize secrets or internal fields.

# Common Mistakes

- Returning an unbounded queryset.
- Filtering detail objects after retrieving them unrestricted.
- Causing writes or expensive external operations during GET.

# Debugging

Check URL pattern, authentication/permission, queryset filters, serializer output, and pagination settings. Use Django query logging or a test query counter for N+1 issues.

# Interview Questions

1. Is GET idempotent? Yes in intended effect; repeat requests do not change state.
2. What does queryset laziness mean? SQL execution is deferred until evaluation.
3. Why can detail lookup return 404 for an existing row? It may be excluded by the scoped queryset.

# Practice Exercises

1. Add pagination and stable ordering to a list endpoint.
2. Write separate tests for collection and missing detail.
3. Optimize a serializer that reads a related category.

# Chapter Summary

GET endpoints turn a permission-scoped queryset into a bounded, stable representation. Treat collection and detail reads separately, and test both content and visibility behavior.

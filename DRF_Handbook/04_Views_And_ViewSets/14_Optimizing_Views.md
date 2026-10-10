# Introduction

Optimizing DRF views means reducing unnecessary database work, serialization cost, and response size while preserving the endpoint contract. Start with measurements, then improve the query and representation path.

# Why this concept exists

A view that works with ten rows can become slow with thousands. Nested serializers, unbounded list endpoints, and repeated ORM queries are common causes.

# Real-world analogy

Optimization is arranging a warehouse picking route after measuring wasted trips, not blindly adding more shelves.

# Internal Working

```text
view queryset -> filtering/order/pagination -> ORM plan -> related fetches
             -> serializer iteration -> response bytes
```

The view controls queryset shape and pagination; serializer fields determine which relations are touched.

# Request Lifecycle

The lifecycle remains the same, but the view should filter before serialization, paginate collections, and fetch needed relations efficiently.

# Syntax

```python
return Order.objects.select_related("customer").prefetch_related("items")
```

# Complete Example 1

Optimize a list endpoint with relation loading and pagination. Assume `OrderSerializer` accesses `customer` and many `items`.

```python
# orders/views.py
from rest_framework import generics
from rest_framework.pagination import PageNumberPagination
from .models import Order
from .serializers import OrderSerializer


class OrderPagination(PageNumberPagination):
    page_size = 25
    page_size_query_param = "page_size"
    max_page_size = 100


class OrderListView(generics.ListAPIView):
    serializer_class = OrderSerializer
    pagination_class = OrderPagination

    def get_queryset(self):
        return (
            Order.objects.filter(owner=self.request.user)
            .select_related("customer")
            .prefetch_related("items")
            .order_by("-created_at")
        )
```

```python
# orders/urls.py
from django.urls import path
from .views import OrderListView

urlpatterns = [path("orders/", OrderListView.as_view(), name="order-list")]
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("orders.urls"))]
```

**Line-by-line:** Pagination bounds page size and caps client selection. The generic list view handles response flow. `get_queryset` filters ownership, joins single-valued customer, prefetches many-valued items, and orders deterministically. URL registers route.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** request -> owner filter -> paginator -> ORM fetch with join/prefetch -> serializer -> response. **Internal working:** `select_related` uses SQL joins for FK/one-to-one; `prefetch_related` performs separate bounded queries for collections and joins in Python. **Professional use case:** Order history. Removing pagination permits huge payloads; removing relation loading can create N+1 queries.

# Complete Example 2

Project only required fields:

```python
queryset = Product.objects.only("id", "name", "price_cents").order_by("id")
```

Use `only()` only when omitted fields will not be accessed by serializer or model methods; deferred accesses can trigger extra queries. Compare with serializer fields and profile.

# Complete Example 3

Test query count for a list:

```python
with self.assertNumQueries(3):
    response = self.client.get(reverse("order-list"))
```

The exact count depends on pagination, authentication, and database behavior. Use this to catch N+1 regressions, but avoid brittle assertions for incidental queries.

# Advanced Example

For heavy counts, choose pagination deliberately. Page-number pagination may issue a count query; cursor pagination requires stable ordering. Use `EXPLAIN` and realistic data to evaluate indexes.

# Under the Hood

QuerySets are lazy until evaluated; serializers iterate them and access fields. Relation access can issue extra SQL. Pagination slices or cursors the queryset before serialization.

# Performance Notes

- Measure endpoint duration and query count.
- Paginate, filter, and project data.
- Use `select_related`/`prefetch_related` based on access shape.
- Avoid expensive SerializerMethodField database calls.

# Best Practices

- Optimize view queryset to serializer.
- Keep ordering stable for pagination.
- Cap client-controlled page sizes.
- Benchmark with representative data.

# Common Mistakes

- Adding relation loading without serializer access.
- Using `only()` then triggering deferred-field queries.
- Returning unbounded list results.
- Assuming ViewSet or generic view is inherently optimized.

# Debugging

Inspect SQL query count, query plan, serializer fields, pagination, and response size. Profile before and after a change.

# Interview Questions

1. What is N+1? Extra query per object for a relation.
2. `select_related` vs `prefetch_related`? SQL join for single-valued vs separate fetch for many-valued.
3. Why paginate? Bound query and response resource usage.
4. What can cursor pagination require? Stable ordering.

# Practice Exercises

1. Add a nested order serializer and measure queries.
2. Optimize with relation loading.
3. Add pagination and a maximum page size.

# Chapter Summary

View optimization is primarily queryset and representation design. Measure first, paginate, load relations efficiently, and keep payloads bounded.

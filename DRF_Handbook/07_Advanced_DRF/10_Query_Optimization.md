# Introduction

Query optimization reduces database round trips, transferred columns, sorting work, and response latency while preserving correct API behavior.

# Why this concept exists

A serializer can trigger hidden database access for every result row; an endpoint that works with ten records may become slow with thousands.

# Real-world analogy

Query optimization is planning one efficient warehouse trip instead of sending a worker back for every item.

# Internal Working

Django QuerySets compose lazy SQL. Evaluate query count and plans; use joins/prefetching, indexes, pagination, and selected fields according to actual serializer access patterns.

# Authentication Flow (where applicable)

Request -> auth/policy -> scoped queryset -> optimized ORM query -> serializer evaluation -> response.

# Syntax

```python
Product.objects.select_related("category").prefetch_related("tags")
```

# Complete Example 1

```python
class ProductListView(ListAPIView):
    serializer_class = ProductSerializer

    def get_queryset(self):
        return (
            Product.objects.filter(active=True)
            .select_related("category")
            .prefetch_related("tags")
            .order_by("name", "pk")
        )
```

The filter applies visibility, `select_related` joins a single-valued category, `prefetch_related` batches many-valued tags, and ordering is deterministic. Match relation loading to serializer fields. Unused prefetching can waste memory and query time.

# Complete Example 2

```python
from django.test.utils import CaptureQueriesContext
from django.db import connection

with CaptureQueriesContext(connection) as queries:
    response = self.client.get("/api/products/")
assert response.status_code == 200
print("query count:", len(queries))
```

The context captures executed SQL during a test request. Use this to compare before/after changes, not as a fixed universal query count when framework/database behavior changes. Avoid printing sensitive SQL parameters in shared logs.

# Complete Example 3

```python
class ProductListView(ListAPIView):
    serializer_class = ProductSerializer
    pagination_class = ProductPagination

    def get_queryset(self):
        return Product.objects.only("id", "name", "price").order_by("pk")
```

`only()` defers other columns, which can save transfer for large rows. Accessing a deferred field later triggers another query per object, so ensure the serializer does not read omitted data. Pagination bounds object creation.

# Advanced Example

Use database `EXPLAIN`/`EXPLAIN ANALYZE` on representative SQL; assess indexes, cardinality, joins, lock contention, and count queries. Optimize query plans while retaining tenant filters and correctness.

# Under the Hood

QuerySet operations construct a query tree; evaluation occurs on iteration, indexing, `list()`, or serializer consumption. Each relation access may trigger SQL unless loaded.

# Performance Notes

Measure end-to-end latency, SQL count/time, rows scanned, serialization CPU, and payload size. N+1 is one common issue, not the only bottleneck.

# Security Notes

Never remove access filters to make queries faster. Ensure raw SQL remains parameterized and optimized plans retain tenant/visibility conditions.

# Best Practices

- Profile with realistic data and traffic.
- Align relation loading with serializer access.
- Paginate and index common filters/order.
- Watch deferred-field access and count-query cost.

# Common Mistakes

- Adding `prefetch_related` everywhere.
- Optimizing query count while fetching excessive data.
- Forgetting indexes and stable ordering.

# Debugging

Inspect SQL, query count, execution plan, serializer fields, and response time breakdown.

# Interview Questions

1. What causes N+1 queries?
2. Difference between `select_related` and `prefetch_related`?
3. Why can `only()` introduce new queries?

# Practice Exercises

1. Measure query count for nested product serialization.
2. Optimize one N+1 endpoint and compare plans.
3. Add appropriate index for a measured filter.

# Chapter Summary

Optimization begins with measurement. Shape scoped querysets to the representation, bound list work, and verify both query plans and correctness after every change.

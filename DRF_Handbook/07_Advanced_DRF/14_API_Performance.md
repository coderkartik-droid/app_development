# Introduction

API performance covers latency, throughput, resource use, and predictable behavior under load.

# Why this concept exists

An endpoint that is fast for one request can become slow or unavailable when traffic, data volume, or dependency latency grows.

# Real-world analogy

Performance engineering is balancing a restaurant's tables, kitchen stations, staff, and supply chain rather than merely making one dish quickly.

# Internal Working

Request time is the sum of network, middleware, authentication, permissions, database, serialization, external calls, and rendering. Measure each component to find the actual bottleneck.

# Authentication Flow (where applicable)

Client -> edge -> DRF policy -> optimized query/service -> serializer -> response; instrumentation records latency and resource costs at each stage.

# Syntax

```python
queryset = Product.objects.select_related("category").order_by("pk")
```

# Complete Example 1

```python
class ProductListView(ListAPIView):
    serializer_class = ProductSerializer
    pagination_class = ProductPagination

    def get_queryset(self):
        return (
            Product.objects.filter(active=True)
            .select_related("category")
            .order_by("name", "pk")
        )
```

Pagination bounds result size, the active predicate narrows data, `select_related` loads a category in one join, and stable order supports predictable pages. Confirm the serializer accesses category and use a measured index; unnecessary joins can slow queries.

# Complete Example 2

```python
from time import perf_counter
import logging

logger = logging.getLogger(__name__)

def timed_operation(operation, request_id):
    start = perf_counter()
    result = operation()
    logger.info("operation_complete", extra={
        "request_id": request_id,
        "duration_ms": (perf_counter() - start) * 1000,
    })
    return result
```

The monotonic performance clock measures elapsed duration. Structured logs include correlation ID and timing, not user secrets. Production should instrument failures too (e.g., `finally`) and use metrics/tracing systems rather than unbounded log cardinality.

# Complete Example 3

```python
response = APIClient().get("/api/products/?page_size=20")
assert response.status_code == 200
assert len(response.data["results"]) <= 20
```

This verifies a response bound. Performance tests should use realistic database volume, inspect query count/latency, and avoid brittle wall-clock assertions in shared CI environments.

# Advanced Example

Set service-level objectives for latency and availability, then use traces to allocate a request budget. Apply caching, indexes, async work, or capacity changes only to measured bottlenecks.

# Under the Hood

Database query evaluation, serializer traversal, Python object allocation, JSON rendering, and network transfer all contribute. A low query count can still produce a slow join or huge payload.

# Performance Notes

Track latency percentiles, error rate, throughput, query time/count, cache hit rate, queue age, and payload sizes.

# Security Notes

Performance controls must not weaken authorization or validation. Rate-limit expensive work and protect metrics endpoints.

# Best Practices

- Measure representative traffic/data.
- Paginate and optimize serializer-driven relations.
- Set timeouts and concurrency bounds on dependencies.
- Avoid unnecessary payload fields and synchronous work.

# Common Mistakes

- Optimizing from local tiny fixtures.
- Adding cache without invalidation/security design.
- Relying on average latency while tail latency is poor.

# Debugging

Use request traces and SQL plans; compare endpoint stages; inspect worker saturation, DB pool, cache, and external dependency timing.

# Interview Questions

1. What is tail latency?
2. Why is query count not sufficient?
3. How do you identify an API bottleneck?

# Practice Exercises

1. Profile a nested list endpoint.
2. Add pagination and compare payload/query costs.
3. Define latency SLO and measurements.

# Chapter Summary

Performance is an end-to-end property. Instrument first, optimize measured bottlenecks, and preserve security and correctness while controlling work at every layer.

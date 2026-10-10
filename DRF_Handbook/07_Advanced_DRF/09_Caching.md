# Introduction

Caching stores reusable results temporarily to reduce repeated computation, database load, and response latency.

# Why this concept exists

Popular read endpoints may repeatedly compute the same result; caching can reuse work while keeping the database available for changing data.

# Real-world analogy

A frequently requested answer is written on a nearby whiteboard so the team does not repeatedly search the archive.

# Internal Working

An API cache key identifies a representation and its relevant inputs. A cache hit returns stored data; a miss executes the view/query and stores the result for a defined time. Authenticated/private content must be varied or isolated by identity and permissions.

# Syntax

```python
from django.views.decorators.cache import cache_page
```

# Complete Example 1

```python
from django.utils.decorators import method_decorator
from django.views.decorators.cache import cache_page
from rest_framework.generics import ListAPIView

@method_decorator(cache_page(60), name="dispatch")
class PublicCatalogView(ListAPIView):
    queryset = Product.objects.filter(active=True).order_by("name", "pk")
    serializer_class = ProductSerializer
    permission_classes = [AllowAny]
```

The decorator caches the response for 60 seconds. This is appropriate only for truly public, representation-stable data; ensure query parameters and relevant headers are included in cache variation. Do not apply blindly to user-specific results.

# Complete Example 2

```python
from django.core.cache import cache

def get_public_category_count():
    key = "catalog:public-category-count:v1"
    value = cache.get(key)
    if value is None:
        value = Category.objects.filter(active=True).count()
        cache.set(key, value, timeout=300)
    return value
```

The function reads a versioned key, computes on miss, and stores for five minutes. `None` is used as miss sentinel; if `None` can be a legitimate cached value, use a unique sentinel. Invalidate or version the key when relevant data changes.

# Complete Example 3

```python
response = APIClient().get("/api/catalog/")
assert response.status_code == 200
```

A response test does not prove cache behavior. Add an integration test that verifies hit/miss or invalidation using the configured cache backend, while avoiding brittle assumptions about backend internals.

# Advanced Example

Use cache-aside with explicit TTL and invalidation/versioning. For high-contention keys, consider request coalescing; for private responses, design cache key and `Vary` semantics to prevent cross-user data leaks.

# Under the Hood

Django cache backends store serialized values in local memory, Redis, Memcached, or other configured infrastructure. Cache consistency is separate from database transactions.

# Performance Notes

Measure hit ratio, latency, memory, and invalidation cost. Caching a fast query may add complexity without benefit.

# Security Notes

Do not cache sensitive responses in shared/public caches. Include tenant/user identity in private keys when caching is justified and set appropriate `Cache-Control` headers.

# Best Practices

- Cache only measured hot paths.
- Define TTL, key version, and invalidation owner.
- Separate public and private response caching.
- Use shared cache in multi-worker production.

# Common Mistakes

- Caching a response without user variation.
- Forgetting invalidation after writes.
- Treating cache contents as durable source of truth.

# Debugging

Inspect cache backend, key, TTL, hit/miss metrics, `Vary` and cache-control headers, and invalidation events.

# Interview Questions

1. What is cache-aside?
2. Why can response caching leak data?
3. What is cache invalidation?

# Practice Exercises

1. Cache a public read endpoint safely.
2. Invalidate/version a key after a write.
3. Demonstrate why caching a user-specific response needs identity variation.

# Chapter Summary

Caching trades freshness and complexity for lower repeated work. Design keys and invalidation deliberately, and never let shared caches mix private representations.

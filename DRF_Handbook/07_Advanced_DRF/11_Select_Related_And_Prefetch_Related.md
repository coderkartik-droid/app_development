# Introduction

`select_related()` and `prefetch_related()` load related model data efficiently when serializers traverse relationships.

# Why this concept exists

Without eager loading, serializing each result can issue additional queries per related object.

# Real-world analogy

`select_related` is joining a single matching file to each record in one trip; `prefetch_related` is collecting a batch of related lists and matching them in memory.

# Internal Working

`select_related` uses SQL joins for foreign key/one-to-one relations. `prefetch_related` runs additional queries and joins results in Python, supporting many-to-many and reverse relations.

# Authentication Flow (where applicable)

Request -> auth/permission -> scoped QuerySet with relation loading -> serializer -> DB result(s) -> response.

# Syntax

```python
Order.objects.select_related("customer").prefetch_related("items")
```

# Complete Example 1

```python
class OrderListView(ListAPIView):
    serializer_class = OrderSerializer

    def get_queryset(self):
        return (
            Order.objects.filter(customer=self.request.user)
            .select_related("customer")
            .order_by("-created_at", "pk")
        )
```

Customer is a single-valued foreign key, so `select_related` joins it into the main query. The user scope is retained. If serializer never reads customer fields, remove the join; if customer is many-valued, use prefetch instead.

# Complete Example 2

```python
class OrderListView(ListAPIView):
    serializer_class = OrderSerializer

    def get_queryset(self):
        return (
            Order.objects.filter(customer=self.request.user)
            .prefetch_related("items__product")
            .order_by("-created_at", "pk")
        )
```

Order items are many-valued and each item references a product; nested prefetch loads those relations in batches. Verify serializer traversal matches this path. Prefetch can consume memory for large unpaginated results, so paginate.

# Complete Example 3

```python
from django.db.models import Prefetch

orders = Order.objects.prefetch_related(
    Prefetch("items", queryset=OrderItem.objects.filter(active=True))
)
```

`Prefetch` customizes the related queryset to load only active items. If the serializer needs inactive items too, the optimization changes output correctness. Add a test asserting representation semantics, not only query count.

# Advanced Example

Use `to_attr` for a filtered prefetch when a distinct in-memory attribute is clearer than populating the relation cache with a subset. Ensure code understands whether it receives a list or relation manager.

# Under the Hood

`select_related` expands the SQL select/join. `prefetch_related` executes after the primary queryset and associates related rows by keys. Prefetch is not one SQL query; it is fewer batched queries.

# Performance Notes

Join width, row duplication, memory, and page size matter. Measure representative query plans and serialized output.

# Security Notes

Eager loading must preserve tenant and visibility rules in both primary and custom prefetch querysets. Do not expose related objects merely because they were loaded.

# Best Practices

- Use joins for single-valued relations, prefetch for collections.
- Load only relations actually accessed.
- Combine with pagination and scoped base querysets.
- Test output and query behavior together.

# Common Mistakes

- Using `select_related` for many-to-many relationships.
- Prefetching unbounded collections.
- Assuming fewer SQL statements always means faster execution.

# Debugging

Inspect serializer relation access, SQL query count, joins, prefetch SQL, and memory/latency under realistic page size.

# Interview Questions

1. Which method uses SQL joins?
2. Which supports many-to-many?
3. Does prefetch mean one query?

# Practice Exercises

1. Optimize order/customer/item serialization.
2. Use a filtered `Prefetch` and verify output.
3. Compare performance and memory for different page sizes.

# Chapter Summary

Eager loading prevents repeated relation queries when aligned with serializer access. Select joins for single-valued relations and prefetch batches for collections, while measuring real costs.

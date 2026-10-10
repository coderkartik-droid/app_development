# Introduction

This chapter reviews common interview questions about DRF CRUD operations and the design choices surrounding them.

# Why this concept exists

Interviewers look for understanding of HTTP semantics, validation, authorization, persistence, and performance—not only memorized class names.

# Real-world analogy

An interview is like reviewing a bridge design: the important point is explaining why each safety mechanism exists and where it can fail.

# Internal Working

Strong answers follow the path from HTTP request through routing, policy, serializer, ORM/database, and response, identifying which layer owns each concern.

# Request Lifecycle

Use this answer framework: identify request intent -> explain DRF dispatch -> describe validation and query scope -> state persistence/status -> mention failure and security cases.

# Syntax

```python
serializer = ProductSerializer(instance, data=request.data, partial=True)
serializer.is_valid(raise_exception=True)
serializer.save()
```

# Complete Example 1

**Q: Explain the difference between PUT and PATCH.**

```python
full = ProductSerializer(product, data=payload)
partial = ProductSerializer(product, data=payload, partial=True)
```

`full` expects a complete writable representation; `partial` permits omitted required fields. Both validate supplied values and update the provided instance. In DRF's generic update view, PUT and PATCH select these modes. Do not say PATCH skips validation.

# Complete Example 2

**Q: How do you prevent users from retrieving one another's objects?**

```python
def get_queryset(self):
    return Product.objects.filter(owner=self.request.user)
```

Generic object retrieval uses this queryset, so out-of-scope IDs behave as not found. Authentication establishes identity; queryset scope and object permissions enforce visibility. This must also apply to list results.

# Complete Example 3

**Q: How do you avoid N+1 queries?**

```python
def get_queryset(self):
    return Product.objects.select_related("category").prefetch_related("tags")
```

`select_related` joins single-valued relations; `prefetch_related` loads collections in separate batched queries. Choose based on serializer access and verify query count rather than applying blindly.

# Advanced Example

**Q: How do you guarantee uniqueness when two creates race?** Validate for a helpful message, add a database unique constraint for integrity, and translate an expected `IntegrityError` at a transaction boundary into a safe API conflict/validation response. A serializer existence check alone is not concurrency-safe.

# Under the Hood

DRF's generic mixins implement common CRUD control flow. Serializer `.is_valid()` creates validated data; `.save()` dispatches to create/update; database constraints and transactions determine durable outcomes.

# Performance Notes

Mention pagination, stable ordering, N+1 prevention, index selection, and measurement in answers about list APIs.

# Best Practices

- Explain tradeoffs, not just names.
- Separate authentication from authorization.
- Include status codes and error handling.
- Tie validation to database integrity.

# Common Mistakes

- Saying ModelViewSet automatically secures CRUD.
- Claiming `PUT` and `PATCH` are interchangeable.
- Ignoring concurrency when discussing uniqueness.

# Debugging

When an answer is challenged, trace one concrete request and name each layer, its responsibility, and the likely failure response.

# Interview Questions

1. What does `ModelViewSet` provide?
2. What does `perform_create` customize?
3. Why should a list endpoint paginate?
4. Why is serializer uniqueness validation insufficient alone?
5. What does a router generate?
6. How does `get_queryset()` support tenant isolation?
7. When choose a custom action over PATCH?
8. When return 201, 204, 400, 403, or 404?

# Practice Exercises

1. Give a two-minute lifecycle explanation for POST.
2. Design an owner-scoped product CRUD API.
3. Explain atomic versus partial bulk import semantics.

# Chapter Summary

Good CRUD answers connect HTTP intent to DRF's lifecycle and database guarantees. Emphasize permission scope, serializer validation, status contracts, and measurable query performance.

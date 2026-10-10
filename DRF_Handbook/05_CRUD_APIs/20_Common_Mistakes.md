# Introduction

CRUD bugs often come from confusing HTTP semantics, trusting client input, or relying on framework defaults without understanding their boundaries.

# Why this concept exists

Recognizing common failure patterns helps prevent data loss, information leaks, duplicate writes, and unstable client integrations.

# Real-world analogy

These mistakes resemble leaving a stockroom unlocked because a visitor has a badge: authentication says who entered; authorization determines what they may access.

# Internal Working

Most failures occur at layer boundaries: URL/action mismatch, parser/input shape mismatch, serializer validation gap, queryset scope gap, transaction gap, or response contract mismatch.

# Request Lifecycle

Debug in order: route -> method/action -> policy -> parsed data -> serializer errors -> object/query scope -> DB outcome -> response status/body.

# Syntax

Avoid this unsafe pattern:

```python
Product.objects.get(pk=request.data["id"])
```

# Complete Example 1

```python
class ProductViewSet(viewsets.ModelViewSet):
    serializer_class = ProductSerializer

    def get_queryset(self):
        return Product.objects.filter(owner=self.request.user)
```

This is the safer lookup baseline for user-owned records: generic detail actions retrieve only rows in the caller's scope. Add authentication and object permissions. If the queryset is simply `Product.objects.all()`, a guessed ID may expose another user's record.

# Complete Example 2

```python
serializer = ProductSerializer(data=request.data, partial=True)
serializer.is_valid(raise_exception=True)
serializer.save()
```

This is another mistake: `partial=True` is intended for an existing instance update, but no instance is passed, so it can permit incomplete creation. Correct update code supplies both the retrieved instance and partial flag. Creation should validate required fields normally.

# Complete Example 3

```python
def get_queryset(self):
    ordering = self.request.query_params.get("ordering", "name")
    allowed = {"name", "-name", "price", "-price"}
    if ordering not in allowed:
        ordering = "name"
    return Product.objects.order_by(ordering)
```

The allowlist prevents arbitrary query fields from becoming ordering input. A stronger API may return a validation error instead of defaulting. The returned query remains an ORM ordering expression, not interpolated raw SQL.

# Advanced Example

Write a regression test for every discovered defect: cross-user object access, invalid PATCH, duplicate create, unstable pagination, and a transaction failure. Fix the root layer instead of adding response-specific patches.

# Under the Hood

DRF generic views centralize object retrieval and serializer validation, but custom overrides can bypass those safeguards. Database constraints catch invariants that application checks cannot guarantee under concurrency.

# Performance Notes

Avoid fixing N+1 with broad data loading that fetches excessive columns/relations. Profile representative requests and verify query counts after changes.

# Best Practices

- Use `get_queryset()` and `get_object()` for scoped access.
- Distinguish PUT from PATCH.
- Validate every client-controlled value.
- Keep tests for permissions, error contracts, and database effects.

# Common Mistakes

- Exposing `__all__` fields without review.
- Catching all exceptions and returning success-shaped data.
- Performing non-idempotent writes on GET.
- Assuming Postman success means automated coverage is unnecessary.

# Debugging

Reproduce one request, inspect response status and `serializer.errors`, trace view dispatch, inspect SQL/transaction state, and test as both authorized and unauthorized users.

# Interview Questions

1. Why is `request.data["owner"]` unsafe? The client can impersonate another owner.
2. What is a common pagination bug? Missing deterministic ordering.
3. Why not catch all exceptions? It hides defects and corrupts error semantics.

# Practice Exercises

1. Identify risks in an unrestricted ModelViewSet.
2. Fix a PATCH endpoint that permits changing owner.
3. Add regression tests for 404/403 access behavior.

# Chapter Summary

Most CRUD mistakes violate a clear boundary: semantics, trust, access, integrity, or resource limits. Trace requests layer by layer and add regression tests that protect the corrected contract.

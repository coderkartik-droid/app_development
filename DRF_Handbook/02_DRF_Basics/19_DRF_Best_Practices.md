# Introduction

DRF best practices help APIs stay predictable, secure, maintainable, and efficient. Good endpoint design starts with a clear contract and ends with tests that exercise the actual HTTP boundary.

# Why this concept exists

API clients depend on behavior over time. Inconsistent schemas, permissive access, or unbounded queries can affect many clients and be difficult to repair after release.

# Real-world analogy

An API contract is a public service promise: stable hours, clear requirements, appropriate access checks, and a receipt that accurately reports the outcome.

# Internal Working

```text
route -> authenticate -> authorize -> parse -> validate
      -> scoped query/transaction -> serialize -> status/render -> test
```

Each layer should have one primary responsibility: routing locates, policies protect, serializers validate/represent, and data access enforces domain persistence rules.

# Request Lifecycle (where applicable)

DRF applies configured parsers, authentication, permissions, throttles, handler logic, exception conversion, and renderer finalization. Production behavior should be tested under the actual settings profile.

# Syntax

```python
class ItemList(generics.ListCreateAPIView):
    serializer_class = ItemSerializer
    permission_classes = [IsAuthenticated]
    queryset = Item.objects.all()
```

# Complete Example 1

A small API view with explicit permission and serializer validation pattern:

```python
from rest_framework import generics
from rest_framework.permissions import IsAuthenticated


class ItemListView(generics.ListCreateAPIView):
    serializer_class = ItemSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Item.objects.filter(owner=self.request.user).order_by("-created_at")

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

Assumes `ItemSerializer` and an `owner` field exist.

**Line-by-line:** Generic list/create view reuses standard GET/POST behavior. `serializer_class` controls validation and representation. `IsAuthenticated` requires an authenticated caller. `get_queryset` limits records to the current owner and orders them. `perform_create` binds ownership from server identity rather than trusting client input. **Flow:** request -> auth -> permission -> owner-scoped queryset or serializer validation -> ORM -> response. **Internal working:** Generic views coordinate standard operations while serializers define data shape. **Professional tip:** Test attempts to read/write another user's data. Removing the owner filter creates an insecure direct-object exposure risk.

# Complete Example 2

Use `select_related` for a frequently accessed foreign key:

```python
def get_queryset(self):
    return (
        Item.objects.filter(owner=self.request.user)
        .select_related("category")
        .order_by("-created_at")
    )
```

The ORM can fetch the single-valued category with a join, reducing per-item queries when serialization reads it. Confirm the serializer actually uses that relation and measure query count.

# Complete Example 3

Test the public contract:

```python
response = self.client.get(reverse("item-list"))
self.assertEqual(response.status_code, 200)
self.assertEqual(set(response.data.keys()), {"count", "next", "previous", "results"})
```

Adjust expected keys to the configured pagination shape. Tests should validate behavior promised to clients without freezing incidental internals.

# Advanced Example

Treat breaking API changes as compatibility work: version intentionally, document deprecation, monitor usage, and test older clients. Use idempotency/concurrency protections for operations where duplicate submissions are costly.

# Under the Hood

Generic views reuse mixins and serializers; they do not replace authorization or query design. Authentication identifies the caller; permissions and queryset filtering constrain the operation/resource.

# Performance Notes

- Paginate collections.
- Avoid N+1 ORM access.
- Restrict serializer fields and nested depth.
- Measure database query count and response size.

# Best Practices

- Default to least privilege.
- Validate all input through serializers.
- Scope querysets to the authenticated principal.
- Return accurate statuses and consistent errors.
- Use migrations and integration tests.

# Common Mistakes

- Exposing every model field.
- Trusting client-provided owner/price/role values.
- Disabling permissions globally for convenience.
- Adding optimization without measuring.

# Debugging

Inspect effective permission/authentication classes, queryset scope, serializer fields, status, query count, and production settings. Reproduce as anonymous and as different users.

# Interview Questions

1. Authentication vs permission? Authentication identifies; permission decides access.
2. Why override `get_queryset`? Apply request-specific filtering and efficient query behavior.
3. Why `perform_create`? Apply server-controlled values during generic creation.
4. How avoid N+1? Use `select_related`/`prefetch_related` aligned with serializer access.

# Practice Exercises

1. Scope an item endpoint by owner.
2. Add pagination and query-count assertions.
3. Test invalid input and cross-user access.

# Chapter Summary

Build DRF APIs around explicit contracts, restrictive permissions, serializer validation, scoped querysets, bounded payloads, and realistic tests. Optimize measured query paths and treat compatibility as a product commitment.

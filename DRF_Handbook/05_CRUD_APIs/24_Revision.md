# Introduction

This chapter is a quick revision of the CRUD API section, from HTTP intent through DRF request processing and production safeguards.

# Why this concept exists

A compact map makes it easier to select the right operation and remember where validation, permissions, querying, and response behavior belong.

# Real-world analogy

Think of a CRUD API as a controlled inventory counter: request method states the task, identity shows who asked, policy limits access, and the database records the result.

# Internal Working

```text
URL/router -> view/action -> authentication/permissions -> parser
           -> serializer/queryset -> model/ORM/database
           -> Response -> renderer
```

# Request Lifecycle

1. Resolve path and HTTP method.
2. Authenticate and authorize.
3. Parse input or read query/path values.
4. Validate serializer input and scope queryset.
5. Read/write through ORM and database constraints.
6. Return documented status, headers, and representation.

# Syntax

| Operation | Verb | DRF behavior |
|---|---|---|
| List | GET collection | Queryset + serializer + pagination |
| Retrieve | GET detail | Scoped object lookup |
| Create | POST collection | Validate then `create()` |
| Replace | PUT detail | Full update (`partial=False`) |
| Modify | PATCH detail | Partial update (`partial=True`) |
| Remove | DELETE detail | Destroy/archive, often 204 |

# Complete Example 1

```python
class ProductViewSet(viewsets.ModelViewSet):
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Product.objects.filter(owner=self.request.user)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

This is the core pattern for an authenticated owner-scoped resource. The queryset governs list and detail visibility; create assigns ownership from authenticated identity. Add pagination, validation, database constraints, and a router. Removing either ownership control creates an exposure or spoofing risk.

# Complete Example 2

```python
class ProductList(generics.ListAPIView):
    queryset = Product.objects.filter(active=True).order_by("name", "pk")
    serializer_class = ProductSerializer
    filter_backends = [filters.SearchFilter, filters.OrderingFilter]
    search_fields = ["name", "sku"]
    ordering_fields = ["name", "price"]
```

The list view uses a visibility filter, deterministic default order, and explicit search/sort allowlists. Configure a paginator and ensure database indexes match common query patterns. Never let a client filter away tenant scope.

# Complete Example 3

```python
serializer = ProductSerializer(instance, data=request.data, partial=True)
serializer.is_valid(raise_exception=True)
serializer.save()
```

This is the essential PATCH serializer sequence: existing instance, partial input, validation, then save. For PUT omit `partial=True`; for create omit the instance. Never call `save()` before validation.

# Advanced Example

For bulk writes or multi-row business actions, specify atomicity and retry semantics, cap input size, use database constraints, and test rollback/conflict behavior. Version breaking response changes and generate OpenAPI documentation.

# Under the Hood

ModelViewSet combines generic view helpers with CRUD mixins. Routers generate routes. Serializers convert between primitive input/output and model instances; the ORM issues parameterized SQL; renderers encode response data.

# Performance Notes

Paginate, stabilize ordering, prevent N+1 queries, add measured indexes, and profile expensive filters/search/counts.

# Best Practices

- Correct method semantics and status codes.
- Auth plus scoped queryset plus object permissions.
- Whitelisted filters, search, and ordering.
- Server-controlled values and database integrity constraints.
- Stable, documented response and error contracts.

# Common Mistakes

- Unbounded list responses.
- Treating PUT as partial.
- Exposing fields or objects by default.
- Relying only on application-level uniqueness checks.

# Debugging

Trace route -> action -> policies -> parsed values -> serializer/queryset -> DB -> response. Inspect errors and SQL; reproduce in API tests.

# Interview Questions

1. What is the PUT/PATCH distinction?
2. Why does `get_queryset()` matter for security?
3. What does a router generate?
4. Why paginate and order list results?
5. Why use DB constraints as well as serializer validation?

# Practice Exercises

1. Build and test a complete owner-scoped resource.
2. Add search, filters, ordering, and bounded pagination.
3. Explain request flow for POST, PATCH, and DELETE.

# Chapter Summary

Reliable CRUD APIs are built from correct HTTP semantics, explicit authorization, serializer validation, scoped ORM queries, database integrity, bounded collection behavior, and a documented response contract.

# Introduction

Production CRUD APIs must be safe, predictable, observable, and efficient—not merely capable of saving model rows.

# Why this concept exists

An API is a long-lived contract consumed by clients beyond the server team's control. Defaults that seem convenient can become security or compatibility failures at scale.

# Real-world analogy

A production API resembles a public service counter: identity is checked, requests are validated, workloads are bounded, actions are recorded, and outcomes follow clear rules.

# Internal Working

Layer responsibilities: URL identifies operation; policy decides access; serializer validates/represents; domain logic enforces workflow; ORM/database enforce persistence and integrity; monitoring exposes failures and load.

# Request Lifecycle

Authenticate -> authorize -> validate -> scope/query -> transactionally change state -> return documented response -> log safe diagnostic context.

# Syntax

```python
class ProductViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Product.objects.filter(owner=self.request.user)
```

# Complete Example 1

```python
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticated
from .models import Product
from .serializers import ProductSerializer

class ProductViewSet(viewsets.ModelViewSet):
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Product.objects.filter(owner=self.request.user).order_by("-created_at", "pk")

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

Authentication identifies the caller, `get_queryset()` restricts every standard lookup to that caller's products, deterministic ordering stabilizes lists, and `perform_create()` sets ownership from trusted identity. Add object-specific permissions and pagination as needed. Removing the queryset scope can create an insecure direct-object reference even when authentication remains enabled.

# Complete Example 2

```python
from django.db import transaction

def place_order(serializer, user):
    with transaction.atomic():
        order = serializer.save(customer=user)
        reserve_inventory(order)
        create_audit_record(order, actor=user)
    return order
```

This illustrates one atomic database boundary for related state changes. The referenced domain functions must use the same database transaction; external payment calls cannot be rolled back by a DB transaction and need an outbox/saga or retry policy.

# Complete Example 3

```python
REST_FRAMEWORK = {
    "DEFAULT_PAGINATION_CLASS": "rest_framework.pagination.PageNumberPagination",
    "PAGE_SIZE": 25,
    "DEFAULT_THROTTLE_CLASSES": [
        "rest_framework.throttling.AnonRateThrottle",
        "rest_framework.throttling.UserRateThrottle",
    ],
    "DEFAULT_THROTTLE_RATES": {"anon": "60/hour", "user": "1000/hour"},
}
```

Pagination and throttling place bounds on response and request volume. Tune rates to the product and infrastructure; DRF throttling is not a replacement for edge-level abuse protection or a distributed rate-limit service.

# Advanced Example

Design idempotent commands for retry-prone writes; use database constraints, transactions, audit trails, structured safe logs, schema validation, and backward-compatible evolution. For external side effects, use durable messaging/outbox patterns.

# Under the Hood

Permission checks occur in request processing; object checks occur when generic views retrieve objects. Database constraints protect concurrent writes and alternate pathways that bypass serializers.

# Performance Notes

Measure query counts and latency, optimize relations, paginate, index measured access paths, and keep serializers from causing N+1 queries.

# Best Practices

- Default to least privilege and request-scoped querysets.
- Validate input and keep server-owned fields read-only.
- Enforce invariants at serializer and database layers.
- Use transactions for coupled writes.
- Document status codes, pagination, errors, and versioning.
- Add tests for authorization, validation, and concurrency-sensitive constraints.

# Common Mistakes

- Treating authentication as authorization.
- Using broad `fields = "__all__"` without review.
- Unbounded list endpoints and unvalidated sorting.
- Swallowing exceptions or exposing stack traces.

# Debugging

Use request IDs, structured logs without secrets, metrics for latency/status/query counts, and tests that reproduce the exact authenticated request.

# Interview Questions

1. Why both queryset scoping and permissions? Scoping limits discoverable objects; permissions express policy including object checks.
2. Why database constraints if serializer validates? Concurrency and alternate write paths.
3. What should production logs omit? Tokens, passwords, and unnecessary personal/sensitive data.

# Practice Exercises

1. Review a ModelViewSet for data exposure and unbounded queries.
2. Add a uniqueness constraint and conflict test.
3. Document the API's retry and deletion semantics.

# Chapter Summary

Production readiness comes from layered controls: scoped access, explicit contracts, bounded work, durable integrity, and observable behavior. Framework defaults help, but do not substitute for design.

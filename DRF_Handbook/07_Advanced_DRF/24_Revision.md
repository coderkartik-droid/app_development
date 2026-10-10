# Introduction

This revision condenses the advanced DRF topics: uploads, response/error design, throttling, caching, query optimization, signals, background jobs, performance, security, testing, logging, and deployment.

# Why this concept exists

Production APIs must remain correct under load, hostile input, retries, and component failures—not just return a valid response once.

# Real-world analogy

A production API is a managed service counter with identity checks, bounded queues, secure storage, operational logs, and a plan for outages.

# Internal Working

```text
TLS/edge -> middleware -> DRF auth/permissions/throttles
-> parser/serializer -> view/service -> DB/cache/storage/worker
-> Response/renderer -> logs/metrics/traces
```

# Syntax

```python
queryset = Order.objects.select_related("customer").prefetch_related("items")
```

# Complete Example 1

```python
class PrivateOrderList(ListAPIView):
    serializer_class = OrderSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return (
            Order.objects.filter(customer=self.request.user)
            .select_related("customer")
            .order_by("-created_at", "pk")
        )
```

The permission requires identity; queryset scoping protects ownership; eager loading supports serializer access; stable order supports pagination. Add pagination and query-count tests. Removing the user filter risks cross-account data exposure.

# Complete Example 2

```python
with transaction.atomic():
    record = Record.objects.create(owner=request.user, state="queued")
    transaction.on_commit(lambda: process_record.delay(record.pk))
```

The record is persisted before task enqueue. `on_commit` avoids a worker seeing rolled-back data but does not eliminate the crash window between commit and publication; use an outbox for guaranteed delivery.

# Complete Example 3

```python
response = APIClient().post("/api/files/", upload, format="multipart")
assert response.status_code in (201, 202, 400, 401, 413)
```

This lists plausible outcomes only for orientation. Real tests assert exact status for a fixed fixture, include authentication, and verify file/state cleanup and ownership.

# Advanced Example

For every endpoint define access policy, payload limits, idempotency/retry rules, transaction boundary, error contract, observability fields, and deployment dependencies.

# Under the Hood

QuerySets are lazy; renderers run after views return; cache/storage/broker state is independent of DB transactions; signals are synchronous unless they explicitly enqueue work.

# Performance Notes

Measure latency percentiles, SQL plans/counts, serialization, cache hit ratio, payload size, worker queue age, and external dependency timeouts.

# Security Notes

Use TLS, secure credentials, tenant-scoped querysets, upload validation, rate limits, redacted logs, strict production settings, and least-privilege infrastructure credentials.

# Best Practices

- Keep public contracts explicit.
- Measure before optimizing.
- Use `on_commit` or outbox for tasks.
- Test boundaries and failure paths.
- Deploy compatible migrations with rollback plans.

# Common Mistakes

- Unbounded uploads, lists, and task queues.
- Caching private responses incorrectly.
- Assuming signals or transactions provide distributed reliability.
- Exposing internal exceptions or secrets.

# Debugging

Trace a correlation ID through edge, DRF, SQL, cache/storage, worker, and response. Check logs, metrics, query plans, and persisted state.

# Interview Questions

1. When use `select_related` versus `prefetch_related`?
2. What failure does a transactional outbox solve?
3. Why is a cache not a source of truth?
4. What is the security risk of an unscoped queryset?

# Practice Exercises

1. Secure and optimize a file-processing endpoint.
2. Add structured logs and request/task correlation.
3. Create deployment checks and rollback plan.

# Chapter Summary

Advanced DRF requires system-level thinking: protect every boundary, bound all work, make side effects retry-safe, optimize from evidence, test failures, and deploy with observability and recovery.

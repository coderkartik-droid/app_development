# Introduction

This chapter reviews advanced DRF architecture, operations, security, and performance questions.

# Why this concept exists

Senior-level interviews look for system reasoning: how pieces behave under concurrency, failure, and scale.

# Real-world analogy

An architect must explain not only what a component does, but how the entire building behaves during load, outage, and repair.

# Internal Working

Answer by tracing request path, ownership boundary, persistence/transaction semantics, failure mode, and how behavior is measured or tested.

# Syntax

```python
transaction.on_commit(lambda: task.delay(record.pk))
```

# Complete Example 1

**Q: How do you avoid N+1 queries in a DRF serializer?**

```python
queryset = Order.objects.select_related("customer").prefetch_related("items")
```

Use `select_related` for single-valued relations and `prefetch_related` for collections. Align the query plan to fields the serializer actually accesses and measure query count plus execution time.

# Complete Example 2

**Q: Why is a database transaction not enough for a database write plus task enqueue?**

```python
with transaction.atomic():
    record = Record.objects.create(...)
    transaction.on_commit(lambda: process_record.delay(record.pk))
```

The DB transaction controls DB state, not broker state. `on_commit` avoids enqueueing for rolled-back rows, but a process crash after commit and before enqueue can lose delivery. A transactional outbox closes that gap.

# Complete Example 3

**Q: How do you design an authenticated upload endpoint?**

```python
parser_classes = [MultiPartParser, FormParser]
permission_classes = [IsAuthenticated]
```

Then validate actual content, cap bytes/dimensions, assign ownership server-side, store privately, scan as appropriate, and authorize downloads. Parser and authentication declarations are only part of the design.

# Advanced Example

**Q: How do you operate an API?** Define SLOs; instrument latency/error/traffic/saturation; correlate logs/traces; use health checks; deploy compatible migrations; manage secrets; back up and rehearse restore; establish incident playbooks.

# Under the Hood

DRF orchestration ends at application response boundaries; database, cache, storage, queues, proxies, and workers each have independent consistency and failure behavior.

# Performance Notes

Explain percentiles, query plans, pagination, caches, payload size, concurrency, and external timeouts rather than saying “add Redis.”

# Security Notes

Discuss threat model, auth versus authorization, IDOR, CSRF, token revocation, rate limits, secrets, and safe logs.

# Best Practices

- Answer with a concrete flow and tradeoffs.
- State assumptions and failure cases.
- Mention tests and observability.

# Common Mistakes

- Overstating guarantees of transactions or JWT.
- Ignoring access scope and list behavior.
- Suggesting optimization without measuring.

# Debugging

For every scenario, identify the responsible layer and evidence needed to prove the issue.

# Interview Questions

1. What is a transactional outbox?
2. How do you secure object access?
3. What is the difference between cache and source of truth?
4. How do you safely deploy schema changes?
5. What is refresh-token reuse detection?
6. How should errors be normalized?

# Practice Exercises

1. Explain an upload pipeline and its threat controls.
2. Design API behavior when a task broker is unavailable.
3. Create a deployment and rollback scenario.

# Chapter Summary

Senior DRF answers explain system boundaries and guarantees, not just APIs. Connect architecture decisions to failure modes, security, measurements, and tests.

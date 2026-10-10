# Introduction

These exercises apply advanced DRF concepts to realistic production concerns.

# Why this concept exists

Hands-on scenarios expose issues that isolated feature demonstrations miss, such as storage/database inconsistency and background retries.

# Real-world analogy

This is a production incident drill: practice the workflow before the system is under real customer pressure.

# Internal Working

For each scenario, specify request path, expected data/side effects, failures, transaction boundary, security checks, and observability.

# Authentication Flow (where applicable)

```text
client -> auth -> permission -> view/service -> DB/cache/storage/task
       -> response -> logs/metrics
```

# Syntax

```python
with transaction.atomic():
    operation()
```

# Complete Example 1

Implement an authenticated file upload with a maximum size:

```python
class BoundedFileField(serializers.FileField):
    def to_internal_value(self, data):
        if data.size > 5 * 1024 * 1024:
            raise serializers.ValidationError("File must be 5 MB or smaller.")
        return super().to_internal_value(data)
```

The field rejects oversize data before normal file-field validation. Configure reverse-proxy request limits too, because application validation happens after bytes may have reached the server. Add ownership, content validation, private storage, and tests at boundary sizes.

# Complete Example 2

Implement task enqueue after commit:

```python
with transaction.atomic():
    report = Report.objects.create(owner=request.user, status="queued")
    transaction.on_commit(lambda: generate_report.delay(report.pk))
```

The report persists before the worker is notified. Exercise: simulate rollback and confirm no task is enqueued; then document the remaining crash gap and design an outbox if delivery is critical.

# Complete Example 3

Add a regression test for private caching:

```python
client.force_authenticate(user=alice)
alice_response = client.get("/api/private-profile/")
client.force_authenticate(user=bob)
bob_response = client.get("/api/private-profile/")
assert alice_response.data != bob_response.data
```

The test checks users receive different representations but does not prove the cache is correctly keyed in every state. Inspect cache headers/key strategy and test sequential requests with caching enabled.

# Advanced Example

Capstone: create a secure document API with tenant scope, multipart upload, content constraints, async malware scan, status endpoint, safe download, structured audit event, rate limits, schema docs, and tests for broker/storage failures.

# Under the Hood

Use APIClient for request lifecycle, unit-test services/permissions, and integration-test storage/broker boundaries with controlled fakes. Each test should verify both output and persisted state.

# Performance Notes

Measure upload memory, query count, task queue age, and endpoint latency under representative workloads.

# Security Notes

Use synthetic credentials/files, test malicious/oversized input, and never point exercises at production infrastructure.

# Best Practices

- Write failure-path tests deliberately.
- Separate test levels and external boundaries.
- Verify cleanup and retry behavior.

# Common Mistakes

- Assuming happy-path tests prove resilience.
- Testing only permission classes but not actual routes.
- Ignoring orphan files/tasks after failures.

# Debugging

Correlate request/task IDs, inspect cache/storage/db state, and reproduce failures at one boundary at a time.

# Interview Questions

1. How would you test an upload that fails after storage save?
2. How can a cache leak private data?
3. Why test worker idempotency?

# Practice Exercises

1. Implement the secure document capstone.
2. Add a failure injection test for task delivery.
3. Review access logs for secret leakage.

# Chapter Summary

Advanced practice should exercise boundaries and failures, not just endpoint success. Define guarantees, test them through appropriate layers, and measure resource behavior.

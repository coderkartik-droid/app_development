# Introduction

Background tasks move slow or retryable work out of the HTTP request process, such as sending email, generating reports, or processing uploads.

# Why this concept exists

Long-running synchronous work ties up web workers, increases timeouts, and makes request reliability depend on external systems.

# Real-world analogy

The API counter accepts a job ticket quickly; a back-office team completes the work and updates its status later.

# Internal Working

The API validates and persists an operation, commits it, then enqueues a task to a broker. A worker executes it, records outcome, and retries according to idempotent failure rules.

# Authentication Flow (where applicable)

Client -> auth/permission -> request validation -> DB transaction/job record -> enqueue after commit -> 202 response -> worker -> DB/status/result.

# Syntax

```python
transaction.on_commit(lambda: generate_report.delay(report.pk))
```

# Complete Example 1

```python
# tasks.py with Celery
from celery import shared_task
from .models import Report

@shared_task(bind=True, max_retries=3)
def generate_report(self, report_id):
    report = Report.objects.get(pk=report_id)
    try:
        report.build_file()
        report.status = "complete"
        report.save(update_fields=["status"])
    except TemporaryReportError as exc:
        raise self.retry(exc=exc, countdown=30)
```

The task receives a primitive ID, reloads current state, and retries only an expected temporary failure. Add terminal failure status handling, idempotency, bounded execution time, and safe logging. Do not pass request objects or credentials to the worker.

# Complete Example 2

```python
from django.db import transaction
from rest_framework import status
from rest_framework.response import Response

def create_report(request):
    report = Report.objects.create(owner=request.user, status="queued")
    transaction.on_commit(lambda: generate_report.delay(report.pk))
    return Response({"id": report.pk, "status": report.status},
                    status=status.HTTP_202_ACCEPTED)
```

The database row is created first; `on_commit` enqueues only after successful commit; 202 communicates asynchronous processing. For reliable delivery across process crashes between commit and enqueue, use a transactional outbox pattern.

# Complete Example 3

```python
response = APIClient().post("/api/reports/", {"kind": "monthly"}, format="json")
assert response.status_code == 202
assert response.data["status"] == "queued"
```

The test verifies the API contract, not worker completion. Unit-test the task separately and integration-test broker behavior when needed. Avoid requiring a live external service in ordinary unit tests.

# Advanced Example

Use a transactional outbox when job delivery must not be lost: write an outbox event in the same DB transaction, then a dispatcher publishes it and marks delivery. Make workers idempotent because task delivery is often at least once.

# Under the Hood

A broker transports messages; workers deserialize task arguments and execute independently of the web process. Queue acknowledgement/retry settings determine delivery guarantees and failure windows.

# Performance Notes

Separate queues by workload, bound concurrency, monitor queue age, and set task timeouts. Avoid placing large file contents in messages; pass storage references.

# Security Notes

Authorize job creation and status access; do not expose another user's job. Keep broker credentials secret and avoid sensitive task payloads.

# Best Practices

- Enqueue after commit.
- Pass IDs/references, not ORM objects or request credentials.
- Design idempotent tasks and explicit retry classes.
- Expose operation status for 202 work.

# Common Mistakes

- Performing lengthy work before returning the API response.
- Retrying permanent errors forever.
- Assuming DB transaction includes broker publication.

# Debugging

Inspect job record, broker queue, worker logs, retry count, task ID, and transaction commit. Correlate API request and task identifiers.

# Interview Questions

1. Why use background tasks?
2. Why enqueue with `on_commit`?
3. What is at-least-once delivery and why idempotency?

# Practice Exercises

1. Move report generation into a task.
2. Add status polling endpoint with owner scope.
3. Design an outbox for reliable enqueue.

# Chapter Summary

Background tasks improve request latency and resilience, but introduce distributed failure. Persist intent, enqueue after commit, make tasks idempotent, and monitor their lifecycle.

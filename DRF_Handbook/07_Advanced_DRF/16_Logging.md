# Introduction

Logging records diagnostic and audit events so teams can understand behavior, investigate failures, and operate an API.

# Why this concept exists

Without useful logs, production incidents become guesswork. Poorly designed logs can also leak credentials and personal information.

# Real-world analogy

Logs are a flight recorder: they capture enough events to reconstruct what happened without recording every passenger's private details.

# Internal Working

Django's logging configuration routes records by logger, level, filters, and handlers to console or centralized systems. Structured fields enable search and correlation across services.

# Authentication Flow (where applicable)

Request ID -> authentication outcome -> authorization decision/event -> view operation -> database/task result -> response status; log safe context at each relevant boundary.

# Syntax

```python
logger = logging.getLogger(__name__)
logger.info("resource_created", extra={"request_id": request_id})
```

# Complete Example 1

```python
import logging
from rest_framework.views import APIView
from rest_framework.response import Response

logger = logging.getLogger(__name__)

class ExportView(APIView):
    def post(self, request):
        export = create_export(request.user)
        logger.info("export_queued", extra={
            "user_id": request.user.pk,
            "export_id": export.pk,
            "request_id": getattr(request, "request_id", None),
        })
        return Response({"id": export.pk, "state": "queued"}, status=202)
```

The logger records a domain event after creation with correlation and safe identifiers. The request must already be authenticated and permission-checked. Configure structured formatting and avoid logging input bodies, tokens, or secrets.

# Complete Example 2

```python
# settings.py
LOGGING = {
    "version": 1,
    "disable_existing_loggers": False,
    "handlers": {"console": {"class": "logging.StreamHandler"}},
    "loggers": {
        "catalog": {"handlers": ["console"], "level": "INFO", "propagate": False},
    },
}
```

This routes the `catalog` logger to console at INFO level. Production should emit structured records to the platform collector, configure rotation/retention centrally, and preserve framework logs appropriately.

# Complete Example 3

```python
response = APIClient().post("/api/exports/", {}, format="json")
assert response.status_code in (202, 400, 401)
```

A test should assert one precise outcome based on its identity/payload. Logging tests can use `assertLogs` for important events but should avoid coupling all business tests to exact log wording.

# Advanced Example

Add correlation IDs at trusted middleware/proxy boundaries, propagate them to background tasks and downstream calls, and pair logs with metrics/traces. Sanitize user-controlled strings to prevent log injection.

# Under the Hood

Python logging creates LogRecords; handlers format and emit them. Async logging or collectors may buffer records, so logs are not a durable audit ledger unless designed as one.

# Performance Notes

High-volume synchronous logging can add latency. Use appropriate levels, structured sampling where safe, and nonblocking centralized collection.

# Security Notes

Redact tokens, passwords, session IDs, sensitive personal data, and payment values. Restrict log access and retention.

# Best Practices

- Log events with stable IDs and severity.
- Correlate request/task IDs.
- Separate operational logs from durable audit records.
- Monitor errors and latency without excessive cardinality.

# Common Mistakes

- Logging full request/response bodies by default.
- Swallowing errors after logging.
- Treating console output as permanent audit storage.

# Debugging

Check logger namespace, level, handler, propagation, collector, redaction filters, and correlation ID continuity.

# Interview Questions

1. Difference between logs, metrics, and traces?
2. Why not log Authorization headers?
3. What is a correlation ID?

# Practice Exercises

1. Add structured event logs for export creation.
2. Add a request ID to API and worker logs.
3. Define a redaction and retention policy.

# Chapter Summary

Useful logs are structured, correlated, and safe. Record enough to diagnose operations while keeping credentials and sensitive data out of log pipelines.

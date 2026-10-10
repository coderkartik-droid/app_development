# Advanced DRF Cheat Sheet

## Key Concepts

- Uploads: parser + serializer + storage policy.
- Errors: DRF exceptions and consistent handler.
- Throttling: fairness/abuse layer, not DDoS shield.
- Caching: keys, TTL, invalidation, privacy.
- Background tasks: durable intent, idempotent retries.
- Production: observability, secure settings, safe migrations.

## Important Syntax

```python
queryset = Order.objects.select_related("customer").prefetch_related("items")
```

```python
with transaction.atomic():
    record = Record.objects.create(...)
    transaction.on_commit(lambda: task.delay(record.pk))
```

## Frequently Used Classes and Methods

`MultiPartParser`, `ImageField`, `exception_handler`, `ScopedRateThrottle`, `cache.get/set`, `transaction.atomic`, `transaction.on_commit`, `select_related`, `prefetch_related`.

## Common Decorators

- `@action(...)` custom ViewSet operation.
- `@receiver(signal, sender=...)` Django signal.
- `@shared_task` Celery task.
- `@method_decorator(cache_page(...), name="dispatch")`.

## HTTP Status Codes

202 accepted async work · 204 no body · 400 invalid · 413 payload too large (often edge) · 429 throttled · 500 unexpected failure.

## Request/Response Flow

`edge/TLS -> auth/policy -> parse/validate -> service/ORM/cache/storage/task -> response -> logs/metrics`.

## Serializer Flow

Multipart file -> parser -> FileField/ImageField validation -> model/storage. Errors -> DRF exception -> configured handler -> stable safe payload.

## View/ViewSet Comparison

Views orchestrate HTTP. Put reusable domain rules in services; use ViewSet for resource CRUD; custom action for domain commands.

## Authentication Flow

Auth establishes identity before permissions. Scope querysets for lists and details.

## JWT Flow

Short access -> resource calls; refresh -> dedicated route; rotation/revocation and secure storage are explicit policies.

## Common Mistakes

- Caching private response across users.
- Enqueueing before DB commit.
- Assuming DB transaction includes broker/storage.
- Unbounded upload/task/list payloads.

## Best Practices

- Measure before optimize.
- Use outbox for reliable event delivery.
- Validate file contents and limits.
- Redact logs; correlate requests/tasks.
- Run deployment checks and compatible migrations.

## Interview Revision

`select_related`: join single relation. `prefetch_related`: batched relation queries. `on_commit`: avoids worker seeing rolled-back data but alone does not eliminate publish crash gap.

## Memory Tricks

- **Cache needs TTL + key + invalidation.**
- **External side effects are not inside DB atomicity.**

## One-Page Summary

Advanced DRF is system design: bounded work, safe data access, measured query plans, reliable async behavior, explicit error contracts, secure operations, tests, and deployment readiness.

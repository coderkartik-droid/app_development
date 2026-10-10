# Introduction

Django signals notify interested receivers when certain framework events occur, such as model saves or deletions. They can decouple side effects but should not hide core business workflows.

# Why this concept exists

Some cross-cutting reactions—such as invalidating a cache after a model change—need to respond to changes from multiple code paths.

# Real-world analogy

A signal is a notification bell: interested teams can react when an event happens, but the bell should not secretly perform the main transaction.

# Internal Working

When a sender emits a signal, connected receivers run synchronously by default in the same process. A receiver exception can affect the originating operation; signals do not automatically create transactions or durable queues.

# Authentication Flow (where applicable)

Client -> authenticated/policy-checked API -> view/service -> model save -> signal receiver -> side effect -> response. The receiver does not replace API authorization.

# Syntax

```python
@receiver(post_save, sender=Product)
def product_saved(sender, instance, created, **kwargs):
    ...
```

# Complete Example 1

```python
# signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import Product

@receiver(post_save, sender=Product)
def invalidate_product_cache(sender, instance, **kwargs):
    cache.delete(f"product:{instance.pk}")
```

The receiver listens only for Product saves and invalidates one cache key. Import the module from `AppConfig.ready()` so registration occurs. Cache invalidation is idempotent; avoid slow external calls or exceptions that unpredictably fail saves.

# Complete Example 2

```python
# apps.py
from django.apps import AppConfig

class CatalogConfig(AppConfig):
    name = "catalog"

    def ready(self):
        from . import signals
```

`ready()` imports the receiver module when Django initializes the app registry. The import must not perform database queries. Configure this AppConfig in `INSTALLED_APPS` where required by the Django version/project pattern.

# Complete Example 3

```python
from django.db import transaction

with transaction.atomic():
    product = Product.objects.create(name="Lamp", sku="L-1")
    transaction.on_commit(lambda: enqueue_index_update(product.pk))
```

`on_commit` schedules the callback only after the transaction commits, preventing a worker from observing a row that later rolls back. The callback should enqueue durable work and handle failures; capturing immutable IDs is safer than relying on mutable in-memory state.

# Advanced Example

Prefer explicit domain service calls for critical behavior such as charging payment or changing order state. Use signals for secondary decoupled reactions, and `on_commit` before external side effects.

# Under the Hood

Signal receivers are process-local registrations, not distributed event subscriptions. Each application process must load them; duplicate registration can occur without stable dispatch IDs.

# Performance Notes

Receivers add hidden synchronous work to every matching operation. Keep them fast and measure; move costly work to a background task.

# Security Notes

Signals execute under the privileges of the originating process. Do not assume an API permission check applies to management commands or jobs that save the same model.

# Best Practices

- Keep receivers small, idempotent, and documented.
- Register receivers explicitly and avoid import-time side effects.
- Use `transaction.on_commit` for post-commit actions.
- Test receiver execution and failure behavior.

# Common Mistakes

- Hiding core business logic in signals.
- Making network calls synchronously from `post_save`.
- Assuming signals fire for bulk operations that bypass model save behavior.

# Debugging

Confirm receiver module import, sender match, duplicate registration, transaction commit, and receiver logs/errors.

# Interview Questions

1. Are Django signals asynchronous by default? No.
2. Why use `on_commit`?
3. Why avoid critical business logic hidden in signals?

# Practice Exercises

1. Invalidate a cache after model save.
2. Enqueue a task after transaction commit.
3. Compare signal behavior for `save()` and `bulk_create()`.

# Chapter Summary

Signals provide process-local event hooks, not a durable event bus. Reserve them for small secondary reactions; make critical business workflows explicit and transactional.

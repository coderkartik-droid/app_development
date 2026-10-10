# Introduction

Production DRF services need coherent practices for API boundaries, persistence, background work, security, testing, and operations.

# Why this concept exists

As teams and traffic grow, undocumented local conventions create inconsistent behavior and operational risk.

# Real-world analogy

Best practices are shared engineering procedures: different teams can maintain the same facility safely and predictably.

# Internal Working

Use clear layer boundaries: views coordinate HTTP, serializers validate/represent, services enforce domain operations, models/database maintain durable invariants, and workers handle asynchronous tasks.

# Syntax

```python
serializer.is_valid(raise_exception=True)
with transaction.atomic():
    instance = serializer.save()
```

# Complete Example 1

```python
class BookingViewSet(ModelViewSet):
    serializer_class = BookingSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Booking.objects.filter(owner=self.request.user).select_related("venue")

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

This combines least privilege, server-side ownership, and eager loading aligned with a likely nested venue field. Add pagination and business conflict validation. Use services/transactions for multi-row booking invariants.

# Complete Example 2

```python
def reserve_booking(booking_data, actor):
    with transaction.atomic():
        booking = Booking.objects.create(**booking_data, owner=actor)
        inventory = Inventory.objects.select_for_update().get(
            venue=booking.venue, date=booking.date
        )
        inventory.reserve_one()
        return booking
```

The transaction and row lock protect a concurrent reservation invariant. Handle unavailable inventory with a domain error; lock order consistently to reduce deadlocks. External notifications should occur after commit.

# Complete Example 3

```python
class BookingSerializer(serializers.ModelSerializer):
    class Meta:
        model = Booking
        fields = ["id", "venue", "date", "status"]
        read_only_fields = ["id", "status"]
```

Explicit fields prevent accidental exposure and keep workflow status server-controlled. Add input-only serializers if create fields differ. Review schema and tests whenever a field is added.

# Advanced Example

Adopt API contracts, idempotency for retry-prone commands, outbox for reliable events, trace propagation, schema CI checks, and staged backward-compatible deployment.

# Under the Hood

DRF handles transport lifecycle; Django ORM and DB handle persistence; caches and brokers have separate consistency models. Do not assume one transaction spans these systems.

# Performance Notes

Paginate, optimize serializer relations, set timeouts, measure query plans, and monitor queue age and tail latency.

# Security Notes

Use TLS, secure secrets, least privilege, object/tenant scoping, CSRF for cookies, and safe logs.

# Best Practices

- Make contracts and status semantics explicit.
- Test success, validation, permission, and concurrency cases.
- Keep writes transactional where coupled.
- Instrument critical operations.
- Patch dependencies and rehearse recovery.

# Common Mistakes

- Overly broad serializers or permissions.
- Unbounded queries and unmeasured caching.
- Hidden side effects in signals.
- Non-idempotent retries.

# Debugging

Follow correlation IDs across request, database, cache, and task processing; compare observed behavior to tests and schema.

# Interview Questions

1. Why separate service logic from view orchestration?
2. How ensure reliable event delivery after database commit?
3. What is an idempotency key?

# Practice Exercises

1. Review a booking API for concurrency and access risks.
2. Add an outbox-backed notification.
3. Define operational SLOs and dashboards.

# Chapter Summary

Production-quality APIs rely on clear boundaries, explicit contracts, strong security, durable invariants, measured performance, and operational readiness.

# Project Overview

Build a multi-tenant service-management API for organizations to manage customers, service requests, assignments, and status transitions. This capstone brings together architecture, auth, validation, testing, and operations.

# Features

- Organization and membership roles.
- Customer and service-request CRUD.
- Assignment and workflow transitions.
- Search, ordering, pagination.
- Audit trail and background notifications.

# Folder Structure

```text
service_api/
  accounts/
  organizations/
  requests/
  common/
  config/
```

# API Endpoints

| Method | Route | Purpose |
|---|---|---|
| GET/POST | `/api/v1/requests/` | List/create requests |
| GET/PATCH | `/api/v1/requests/{id}/` | Retrieve/edit permitted fields |
| POST | `/api/v1/requests/{id}/assign/` | Assign staff |
| POST | `/api/v1/requests/{id}/transition/` | Validated state change |
| GET | `/api/v1/audit-events/` | Admin audit view |

# Database Design

Organization has memberships to users; service request belongs to organization and creator; assignment records assignee; audit event stores actor, action, target ID, and timestamp. Unique constraints prevent duplicate membership and idempotency records.

# Models

```python
class Organization(models.Model):
    name = models.CharField(max_length=160)

class Membership(models.Model):
    organization = models.ForeignKey(Organization, on_delete=models.CASCADE)
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    role = models.CharField(max_length=24)
    class Meta:
        constraints = [models.UniqueConstraint(
            fields=["organization", "user"], name="unique_org_member")]

class ServiceRequest(models.Model):
    organization = models.ForeignKey(Organization, on_delete=models.PROTECT)
    created_by = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.PROTECT)
    title = models.CharField(max_length=180)
    description = models.TextField()
    status = models.CharField(max_length=24, default="open")
    created_at = models.DateTimeField(auto_now_add=True)
```

# Serializers

```python
class ServiceRequestSerializer(serializers.ModelSerializer):
    class Meta:
        model = ServiceRequest
        fields = ["id", "title", "description", "status", "created_at"]
        read_only_fields = ["id", "status", "created_at"]

class TransitionSerializer(serializers.Serializer):
    target = serializers.ChoiceField(choices=["open", "in_progress", "resolved"])
```

# Views

```python
class ServiceRequestViewSet(ModelViewSet):
    serializer_class = ServiceRequestSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return ServiceRequest.objects.filter(
            organization__membership__user=self.request.user
        ).distinct().order_by("-created_at", "pk")

    def perform_create(self, serializer):
        membership = Membership.objects.get(user=self.request.user)
        serializer.save(organization=membership.organization, created_by=self.request.user)
```

# URLs

```python
router = SimpleRouter()
router.register("requests", ServiceRequestViewSet, basename="service-request")
urlpatterns = router.urls
```

# Authentication

Use a project-selected session or JWT mechanism, short credential lifetimes where appropriate, secure refresh strategy, and throttled login/refresh routes.

# Permissions

Organization membership scopes data. Add role-based policy for assignment, transition, and audit endpoints. If users can belong to multiple organizations, require an explicit organization context and filter membership accordingly; never use an arbitrary membership `.get()`.

# Testing

Test tenant isolation, role matrix, invalid transitions, duplicate/retried commands, search/pagination, audit writes, and transaction rollback. Add API schema checks and query-count tests.

# Deployment

Use production settings, managed secrets, PostgreSQL, shared cache, object storage, background workers, TLS, database backups, health checks, centralized observability, and staged migrations.

# Complete Source Code

The snippets show principal contracts. A transition service should lock the row, validate allowed state changes, persist audit in the same transaction, then enqueue notifications after commit:

```python
@transaction.atomic
def transition_request(request_id, actor, target):
    item = ServiceRequest.objects.select_for_update().get(pk=request_id)
    assert_membership(actor, item.organization)
    allowed = {"open": {"in_progress"}, "in_progress": {"resolved"}}
    if target not in allowed.get(item.status, set()):
        raise ValidationError({"target": "Transition is not allowed."})
    previous = item.status
    item.status = target
    item.save(update_fields=["status"])
    AuditEvent.objects.create(actor=actor, action="status_changed",
                              target_id=item.pk, details={"from": previous, "to": target})
    transaction.on_commit(lambda: notify_request_changed.delay(item.pk))
    return item
```

# Line-by-Line Explanation

- Membership creates tenant identity and role.
- Service request status is read-only in ordinary CRUD.
- Transition input limits target states.
- Queryset prevents cross-organization access.
- Create derives organization/actor server-side.
- Transaction service locks the row, checks membership and state, records audit, and queues after commit.

**Flow:** Client -> authentication -> tenant/role policy -> scoped view -> serializer/service -> locked DB transaction -> audit/outbox/task -> response. Every custom action must apply the same scope.

# Best Practices

- Separate ordinary CRUD from state-machine commands.
- Persist audit events for sensitive changes.
- Use row locks and idempotency for concurrent operations.
- Instrument latency, failures, and queue lag.

# Common Mistakes

- Allowing arbitrary status PATCH.
- Selecting the first membership when users have multiple organizations.
- Sending notifications before database commit.

# Improvements

Add SLA deadlines, assignment history, attachments, webhooks, cursor pagination, and outbox-based event delivery.

# Challenge Version

Deploy a horizontally scaled version with tenant-aware caching, idempotent webhooks, schema compatibility checks, and a tested blue/green migration plan.

# Summary

The capstone uses tenant-aware access, role checks, state transitions, transactions, audit, async work, and operational controls to model a production API.

# Introduction

DRF view best practices keep routing, view orchestration, serialization, authorization, and persistence aligned. The right abstraction should make the endpoint's supported operations clear.

# Why this concept exists

Views are the meeting point of many framework features. Poor structure can hide permission gaps, duplicate work, or bypass standard pagination and error behavior.

# Real-world analogy

A well-designed service desk has a clear scope, a repeatable procedure, an access policy, and a concise handoff to data specialists.

# Internal Working

```text
route/action -> authentication -> permission -> scoped queryset
             -> serializer -> service/transaction -> response
```

# Request Lifecycle

The view participates in APIView dispatch and calls helpers in predictable order. Prefer framework hooks that preserve this lifecycle.

# Syntax

```python
def get_queryset(self):
    return Order.objects.filter(owner=self.request.user)
```

# Complete Example 1

An owner-scoped ViewSet with explicit serializer and policy:

```python
# orders/views.py
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticated
from .models import Order
from .serializers import OrderSerializer


class OrderViewSet(viewsets.ModelViewSet):
    serializer_class = OrderSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return (
            Order.objects.filter(owner=self.request.user)
            .select_related("customer")
            .prefetch_related("lines")
            .order_by("-created_at")
        )

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

```python
# orders/urls.py
from rest_framework.routers import SimpleRouter
from .views import OrderViewSet

router = SimpleRouter()
router.register("orders", OrderViewSet, basename="order")
urlpatterns = router.urls
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("orders.urls"))]
```

**Line-by-line:** ModelViewSet supplies CRUD; serializer contract is explicit elsewhere; IsAuthenticated gates requests; queryset filters owner and loads related data efficiently; perform_create sets owner from identity; router generates routes.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** router action -> auth/permission -> scoped queryset -> serializer validation/output -> ORM -> response. **Internal execution:** list queries are optimized according to serializer relations. **Professional use case:** Tenant-owned orders. Removing owner filter can leak data; removing permission can make assumptions about anonymous user unsafe.

# Complete Example 2

Use generic views for a narrow method set:

```python
class OrderList(generics.ListAPIView):
    serializer_class = OrderSerializer

    def get_queryset(self):
        return Order.objects.filter(owner=self.request.user)
```

This exposes only list behavior rather than full CRUD. Choose based on actual endpoint contract, not on minimizing lines.

# Complete Example 3

Use a service for complex operation:

```python
def perform_create(self, serializer):
    create_order_for_user(
        user=self.request.user,
        validated_data=serializer.validated_data,
    )
```

The service can own transaction and domain invariants. Avoid calling `serializer.save()` as well if the service already persists; clearly define the serializer/view/service contract.

# Advanced Example

Apply action-specific serializers/permissions with an explicit fallback. For custom actions, validate payloads, keep transaction boundaries clear, and make duplicate/retry behavior safe where relevant.

# Under the Hood

Generic views and ViewSets combine APIView dispatch, generic helpers, and mixins. Hooks like `get_queryset` and `perform_create` are called at known stages; overriding broad handlers can bypass them.

# Performance Notes

Paginate lists, optimize relation access, reduce payload fields, and avoid repeated queries in method fields.

# Best Practices

- Choose FBV/APIView/generic/ViewSet by complexity and operation set.
- Scope querysets to caller and tenant.
- Use serializer validation and database constraints.
- Prefer hooks over rewriting generic handlers.
- Test permissions, status, query count, and response shape.

# Common Mistakes

- Using ModelViewSet when only read methods are intended.
- Treating router routes as access policy.
- Bypassing `get_object` permission checks.
- Overriding handlers without preserving standard headers/pagination.

# Debugging

Trace URL/action, effective permissions, queryset, serializer, save hook, queries, and response. Use tests for unauthorized and invalid access.

# Interview Questions

1. When use APIView vs generic? Custom flow vs conventional resource operations.
2. Why scope queryset? Enforces visibility for lists and detail lookups.
3. Why prefer `perform_create`? Customizes save while preserving create mixin flow.
4. Does router secure routes? No.

# Practice Exercises

1. Build equivalent list endpoints with APIView and ListAPIView.
2. Scope a ViewSet to current user.
3. Disable unintended CRUD operations and test 405 responses.

# Chapter Summary

DRF views should make action scope, permissions, data visibility, and serializer flow explicit. Reuse framework hooks, optimize queries based on representation, and verify the contract with endpoint tests.

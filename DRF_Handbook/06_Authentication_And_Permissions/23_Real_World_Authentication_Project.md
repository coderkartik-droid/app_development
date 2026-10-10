# Introduction

This project outlines a secure order API in which customers see their orders, support staff can inspect orders, and administrators can manage sensitive workflows.

# Why this concept exists

Authentication becomes meaningful when applied to realistic identity, role, ownership, token, and lifecycle requirements.

# Real-world analogy

An online shop checks who signed in, shows each customer their own receipts, and grants support staff narrowly defined lookup tools.

# Internal Working

```text
login -> short-lived access + refresh
request -> JWT authentication -> role/tenant policy -> scoped order lookup
       -> serializer/service/database -> minimal response
```

# Authentication Flow (where applicable)

Customer obtains tokens, sends access token to APIs, refreshes through a dedicated route, and revokes refresh/session state on logout according to policy.

# Syntax

```http
GET /api/v1/orders/
Authorization: Bearer <access-token>
```

# Complete Example 1

```python
# models.py
from django.conf import settings
from django.db import models

class Order(models.Model):
    class Status(models.TextChoices):
        PENDING = "pending", "Pending"
        PAID = "paid", "Paid"
        SHIPPED = "shipped", "Shipped"

    customer = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.PROTECT)
    status = models.CharField(max_length=20, choices=Status.choices, default=Status.PENDING)
    total = models.DecimalField(max_digits=10, decimal_places=2)
    created_at = models.DateTimeField(auto_now_add=True)
```

The model references the configured user model and protects customer history from accidental user deletion. Status is constrained to known values, and total uses decimal precision. In a real system, calculate total from validated line items rather than trusting client input.

# Complete Example 2

```python
# serializers.py and permissions.py
from rest_framework import serializers
from rest_framework.permissions import BasePermission
from .models import Order

class OrderSerializer(serializers.ModelSerializer):
    class Meta:
        model = Order
        fields = ["id", "status", "total", "created_at"]
        read_only_fields = fields

class IsSupportOrAdmin(BasePermission):
    def has_permission(self, request, view):
        user = request.user
        return bool(user and user.is_authenticated
                    and (user.is_staff or user.groups.filter(name="Support").exists()))
```

The serializer prevents callers from writing status/total directly through this representation. The permission allows staff or support group members; it does not decide which tenant's order they may see. Add explicit support scope and audit access.

# Complete Example 3

```python
# views.py and urls.py
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticated
from rest_framework.routers import SimpleRouter
from .models import Order
from .serializers import OrderSerializer

class OrderViewSet(viewsets.ReadOnlyModelViewSet):
    serializer_class = OrderSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        user = self.request.user
        qs = Order.objects.select_related("customer")
        if user.is_staff or user.groups.filter(name="Support").exists():
            return qs.order_by("-created_at", "pk")
        return qs.filter(customer=user).order_by("-created_at", "pk")

router = SimpleRouter()
router.register("orders", OrderViewSet, basename="order")
urlpatterns = router.urls
```

This read-only viewset prevents accidental generic writes. Customers are scoped to their own orders; support/staff visibility is explicit; `select_related` avoids a likely customer N+1 if serializer later includes customer data. Add pagination and audit logging for support access. Avoid broad staff access if the business requires tenant limits.

# Advanced Example

Implement order creation and state transitions through dedicated serializers/services. Calculate totals server-side, use transactions for order plus line items, record actor and timestamps, and integrate payment providers through idempotent asynchronous workflows.

# Under the Hood

JWT authentication resolves the user, permissions gate the view, `get_queryset()` shapes visible records, serializer produces output, and database constraints protect relationships. Token refresh/revocation settings govern credential lifecycle separately from order permissions.

# Security Notes

Do not expose payment secrets/card data. Use TLS, least privilege, short access lifetime, secure refresh storage, rate limits, safe audit logs, and data-retention policy.

# Best Practices

- Separate customer, support, and admin capabilities.
- Scope records by ownership/tenant.
- Keep financial and workflow fields server-controlled.
- Audit privileged reads and writes.
- Test token and object access boundaries.

# Common Mistakes

- Letting customers set `status` or `total`.
- Giving all support users unrestricted access without audit.
- Treating a signed JWT role claim as permanently current.

# Debugging

Trace token validation, user/group state, selected queryset branch, serializer read-only fields, and database status. Inspect safe audit event metadata.

# Interview Questions

1. Why use ReadOnlyModelViewSet here?
2. How do support users differ from customers?
3. Why calculate order total on server?

# Practice Exercises

1. Add a support-only order lookup with audit records.
2. Implement a validated pending-to-paid transition.
3. Test customer cross-order access and refresh-token logout.

# Chapter Summary

A realistic authentication project combines credentials, roles, ownership scope, read-only financial fields, audited privileged access, and lifecycle tests. Security must hold across every route, not only login.

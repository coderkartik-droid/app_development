# Introduction

Securing a DRF API requires layered defenses across identity, authorization, input validation, transport, storage, operations, and monitoring.

# Why this concept exists

APIs are directly callable by arbitrary clients. Hiding a button in a UI or relying on expected client behavior does not protect the server.

# Real-world analogy

A secure facility uses identity checks, room-specific access, locked storage, visitor logs, alarms, and emergency procedures—not one front-door lock.

# Internal Working

Security controls should be applied at the correct boundary: TLS at transport, authentication at request identity, permissions/querysets at access, serializers at input, database constraints at integrity, and observability at operations.

# Authentication Flow (where applicable)

```text
TLS -> credential validation -> permission/query scope -> input validation
-> domain/database constraints -> safe response -> monitoring/audit
```

# Syntax

```python
REST_FRAMEWORK = {
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
}
```

# Complete Example 1

```python
class OrderViewSet(ModelViewSet):
    serializer_class = OrderSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Order.objects.filter(customer=self.request.user)

    def perform_create(self, serializer):
        serializer.save(customer=self.request.user)
```

The view requires identity, limits reads and updates to the caller's orders, and assigns customer server-side. Serializer fields should keep customer/status/payment metadata read-only. Add object policy, throttling, audit behavior, and tests before production.

# Complete Example 2

```python
class OrderSerializer(serializers.ModelSerializer):
    class Meta:
        model = Order
        fields = ["id", "status", "total", "created_at"]
        read_only_fields = ["id", "status", "total", "created_at"]
```

The representation exposes only selected fields and prevents clients from setting server-calculated values. A separate input serializer may allow address or line items. Never include password hashes, secrets, or internal moderation data.

# Complete Example 3

```python
response = APIClient().get("/api/orders/999999/")
assert response.status_code in (401, 403, 404)
```

This is not a final assertion; tests should authenticate different users and specify the chosen contract. Include cross-user ID access, anonymous access, invalid payloads, and rate-limit behavior.

# Advanced Example

Use threat modeling: identify assets, actors, entry points, trust boundaries, abuse cases, and controls. Add dependency updates, secret management, backups, incident response, and security testing to the deployment lifecycle.

# Under the Hood

DRF runs authentication, permissions, throttling, parsing, and view logic in a defined sequence. The database remains the final integrity boundary for uniqueness and relational constraints.

# Security Notes

Use HTTPS; secure cookies and CSRF for session auth; safe CORS; strong secret management; least privilege; rate limits; minimal error details; dependency patching; and redacted logs.

# Best Practices

- Deny by default.
- Scope every queryset to caller/tenant.
- Validate and constrain client input.
- Enforce invariants in database.
- Monitor authentication failures and anomalous access.

# Common Mistakes

- Treating CORS as authorization.
- Exposing debug mode or detailed exceptions.
- Logging tokens or personal data.
- Relying on UI restrictions or obscure IDs.

# Debugging

Reproduce as the relevant identity, inspect policy and queryset, review sanitized logs, check proxy/TLS/CORS configuration, and verify database constraints.

# Interview Questions

1. Is CORS an access-control mechanism? No, it is a browser cross-origin policy.
2. Why scope querysets? Prevent unauthorized resource enumeration/access.
3. What is defense in depth?

# Practice Exercises

1. Threat-model a multi-tenant order API.
2. Add tests for cross-tenant detail and list access.
3. Create a production security checklist for token handling.

# Chapter Summary

API security is layered. Authentication is only one control; robust systems also enforce authorization, scoped data access, validated input, database integrity, transport security, safe operations, and observability.

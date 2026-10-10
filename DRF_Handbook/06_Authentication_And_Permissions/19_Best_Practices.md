# Introduction

Authentication and permissions are security boundaries, not optional view decoration. A production design must manage identity credentials, enforce least privilege, and make access behavior testable.

# Why this concept exists

The API is directly accessible to clients outside the server team's control. Safe design must assume callers can alter URLs, headers, bodies, and request order.

# Real-world analogy

A secure office checks identity, grants only necessary room access, expires badges, records sensitive entry, and has a revocation process.

# Internal Working

The request pipeline should authenticate first, apply request-level policy, scope data access, validate input, check object rules, and rely on database integrity for durable invariants.

# Authentication Flow (where applicable)

```text
TLS -> credential validation -> view policy -> queryset/object policy
-> validated serializer -> database -> minimal response -> safe audit
```

# Syntax

```python
permission_classes = [IsAuthenticated]

def get_queryset(self):
    return Resource.objects.filter(tenant=self.request.user.tenant)
```

# Complete Example 1

```python
class InvoiceViewSet(ModelViewSet):
    serializer_class = InvoiceSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Invoice.objects.filter(account__members=self.request.user)

    def perform_create(self, serializer):
        serializer.save(created_by=self.request.user)
```

Identity is required, membership limits visible invoices, and creator is server assigned. Add role or object policy for writes and avoid duplicate creation when membership joins can duplicate query rows (use `.distinct()` if needed). Do not trust account IDs from the submitted body.

# Complete Example 2

```python
class IsInvoiceApprover(BasePermission):
    def has_object_permission(self, request, view, obj):
        return request.user.has_perm("billing.approve_invoice") and obj.status == "pending"
```

The permission checks a trusted Django permission and current workflow state. It must be combined with authentication and an appropriate queryset. For an approval operation, a named action/service should also validate the transition atomically to prevent races.

# Complete Example 3

```python
def test_other_tenant_cannot_read_invoice(self):
    self.client.force_authenticate(user=self.other_tenant_user)
    response = self.client.get(f"/api/invoices/{self.invoice.pk}/")
    self.assertEqual(response.status_code, 404)
```

The test proves an out-of-scope object is hidden for one route. Also test list leakage and all mutating methods. `force_authenticate` tests authorization but does not test token/session verification.

# Advanced Example

Maintain a threat model and credential lifecycle: issuance, storage, expiration, rotation, revocation, account disablement, and incident response. For JWT role claims, define how role changes become effective before token expiry.

# Under the Hood

Permission classes execute during DRF initialization; object permission checks are triggered by generic object retrieval. Queryset filtering governs list operations and often intentionally converts forbidden IDs into 404.

# Security Notes

Use HTTPS, secure secret storage, CSRF for cookie credentials, rate limits, safe logs, dependency patching, and minimal response data. Security settings must be verified in deployment, not only development.

# Best Practices

- Deny by default and opt public endpoints in.
- Use scoped querysets and server-assigned ownership.
- Keep credentials short-lived and revocable where practical.
- Add tests for role, ownership, tenant, and anonymous boundaries.
- Monitor failures without logging secrets.

# Common Mistakes

- Using authentication as the sole access check.
- Trusting roles/owners from JSON.
- Assuming token logout invalidates all issued access tokens.
- Treating CORS or UUIDs as authorization.

# Debugging

Reproduce with a specific user, trace authentication then permission and queryset, inspect sanitized logs, and check token age/configuration and database state.

# Interview Questions

1. What is least privilege?
2. Why are querysets part of authorization?
3. How should JWT role changes take effect?

# Practice Exercises

1. Audit an API for object-level exposure.
2. Draft a credential rotation/revocation policy.
3. Build a role and tenant test matrix.

# Chapter Summary

Security best practices combine strong credential handling, least privilege, queryset and object scoping, server-side trusted values, integrity constraints, and tests for adversarial requests.

# Introduction

Authentication and permission defects often arise from confusing identity with authority or assuming that the frontend will behave honestly.

# Why this concept exists

A small policy gap can expose every user record, allow privilege escalation, or leave stolen credentials usable.

# Real-world analogy

A badge that proves someone works for a company does not mean they may open every safe in the building.

# Internal Working

Security review should trace credential validation, permission hooks, queryset scope, client-controlled fields, token lifecycle, and response/log handling.

# Authentication Flow (where applicable)

Client credential -> authenticator -> request identity -> permissions -> scoped object -> view/serializer/database -> response.

# Syntax

Unsafe:

```python
owner = User.objects.get(pk=request.data["owner_id"])
```

# Complete Example 1

```python
class NoteViewSet(ModelViewSet):
    permission_classes = [IsAuthenticated]
    serializer_class = NoteSerializer

    def get_queryset(self):
        return Note.objects.filter(owner=self.request.user)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

This corrects two common mistakes: authenticated users cannot enumerate other users' notes, and create ownership comes from authenticated identity. `IsAuthenticated` alone does not provide either property. Test cross-user list/detail and owner spoofing.

# Complete Example 2

```python
def has_object_permission(self, request, view, obj):
    return bool(
        request.user and request.user.is_authenticated
        and obj.owner_id == request.user.pk
    )
```

The explicit guard avoids unsafe assumptions about anonymous identity. This hook only helps when the view invokes object checks; it does not filter list rows. Pair it with a scoped queryset and suitable request-level permission.

# Complete Example 3

```python
class PublicCatalogView(ListAPIView):
    permission_classes = [AllowAny]
    queryset = Product.objects.filter(is_public=True)
```

The public route intentionally allows anonymous reads but still filters to published public records. `AllowAny` does not mean “return every row.” Add pagination and fields that do not expose internal data.

# Advanced Example

Audit for credential leakage in logs, token storage, browser storage, analytics, and exception reports. Test token revocation, account disablement, refresh reuse, CSRF, and rate-limit behavior—not merely valid login.

# Under the Hood

DRF's default permission behavior can be configured globally; a per-view declaration replaces that list. Custom views can bypass generic object permission hooks if they retrieve directly.

# Security Notes

Never reveal credentials in code/examples, log bearer headers, trust client role claims, or deploy with debug error pages enabled.

# Best Practices

- Explicitly mark public endpoints.
- Use `get_queryset()` for tenant/owner scope.
- Set immutable fields server-side.
- Keep revocation and expiry semantics honest.

# Common Mistakes

- Authentication without authorization.
- Object permission without list filtering.
- Disabling CSRF for cookie-based APIs.
- Treating token deletion on a client as revocation.

# Debugging

Inspect effective view settings, auth class order, request identity, scope filters, object hooks, serializer writable fields, and response status.

# Interview Questions

1. Why is `IsAuthenticated` insufficient for owner-only data?
2. Does a UUID prevent insecure direct object access?
3. Why should authorization tests include list routes?

# Practice Exercises

1. Fix an unrestricted user-owned ModelViewSet.
2. Identify all client-writable trusted fields.
3. Test CSRF and revoked-token cases.

# Chapter Summary

Avoid security assumptions. Authentication, authorization, data scope, credential lifecycle, and trusted field assignment each need explicit implementation and tests.

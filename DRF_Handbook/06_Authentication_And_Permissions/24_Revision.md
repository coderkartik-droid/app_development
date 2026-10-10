# Introduction

This revision summarizes authentication mechanisms, permission boundaries, JWT lifecycle, and production API security.

# Why this concept exists

Security becomes reliable when identity, access policy, credential lifecycle, and data visibility are treated as distinct responsibilities.

# Real-world analogy

Authentication is a verified badge; authorization is the list of rooms that badge holder may enter.

# Internal Working

```text
Client -> Authentication -> request.user/request.auth
       -> permission/queryset/object check -> view
       -> serializer/database -> response
```

# Authentication Flow (where applicable)

Session credentials resolve server-side session state; Basic credentials are resent per request; DRF tokens identify a user through a token record; JWT validates signed claims and usually uses a separate refresh exchange.

# Syntax

```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework_simplejwt.authentication.JWTAuthentication",
    ],
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
}
```

# Complete Example 1

```python
class PrivateNoteViewSet(ModelViewSet):
    serializer_class = NoteSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Note.objects.filter(owner=self.request.user)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

Authentication supplies identity, `IsAuthenticated` rejects anonymous requests, the queryset scopes reads and updates, and `perform_create` assigns a trusted owner. Removing queryset scope can expose another user's records; removing server-side assignment enables ownership spoofing.

# Complete Example 2

```python
class IsOwner(BasePermission):
    def has_object_permission(self, request, view, obj):
        return bool(request.user.is_authenticated
                    and obj.owner_id == request.user.pk)
```

This object rule authorizes a retrieved object, but list views still need queryset filtering. Pair it with authentication and test anonymous behavior rather than assuming `request.user` is always a real user.

# Complete Example 3

```text
Access token: short-lived; sent to resource API.
Refresh token: longer-lived; sent only to refresh endpoint.
Logout: revoke server-side state where supported; client deletion alone is not revocation.
```

Clients should store credentials using a platform-appropriate secure mechanism, replace rotated refresh tokens, and clear local credentials after refresh failure. These lifecycle rules must match server configuration.

# Advanced Example

Combine HTTPS, short token lifetimes, refresh rotation/revocation, least-privilege roles, tenant-scoped querysets, audit trails, rate limits, and tests for account/role changes.

# Under the Hood

DRF authenticates before checking permissions. Generic detail lookup invokes object checks; list access is controlled by the queryset. JWT signatures provide integrity, not confidentiality.

# Security Notes

Never trust client-supplied owner/role values, log credentials, disable CSRF for cookie auth, or rely on unguessable IDs as authorization. Use explicit revocation and key management.

# Best Practices

- Distinguish authentication from authorization.
- Deny by default and opt public endpoints in.
- Scope all user/tenant-owned querysets.
- Test real authentication separately from forced-user permission tests.
- Document logout and token-expiry guarantees.

# Common Mistakes

- Assuming `IsAuthenticated` enforces ownership.
- Assuming object permissions filter lists.
- Treating JWT as encrypted or immediately revocable.
- Using Basic auth without TLS.

# Debugging

Trace credential -> identity -> effective permissions -> queryset/object -> serializer/database -> response. Inspect safe logs and test with multiple identities.

# Interview Questions

1. Compare session, Basic, token, and JWT auth.
2. What do the two BasePermission hooks do?
3. Why scope querysets if object permissions exist?
4. What is refresh-token rotation?
5. Why is logout difficult for stateless access tokens?

# Practice Exercises

1. Secure a multi-tenant resource API.
2. Test owner, non-owner, anonymous, and support roles.
3. Implement and test refresh rotation and logout.

# Chapter Summary

Secure DRF APIs establish identity with an appropriate authenticator, enforce least privilege through permissions and scoped querysets, protect credential lifecycles, and validate those controls with adversarial tests.

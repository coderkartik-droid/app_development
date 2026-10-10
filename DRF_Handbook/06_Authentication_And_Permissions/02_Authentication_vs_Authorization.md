# Introduction

Authentication verifies identity; authorization decides whether that identity may perform an action on a resource. A secure DRF API needs both, often at request and object levels.

# Why this concept exists

Knowing a caller's identity is not enough to decide whether they may read another user's invoice or publish an article.

# Real-world analogy

A passport verifies who you are. A boarding pass and seat assignment determine where you may go.

# Internal Working

Authentication populates `request.user`/`request.auth`. DRF then checks view-level permissions; generic detail views additionally run object-level checks after retrieving an object. Queryset scoping limits which objects can be retrieved or listed.

# Authentication Flow (where applicable)

```text
Client -> Authentication -> identity -> permission policy -> view
       -> scoped queryset/object check -> serializer/database -> response
```

# Syntax

```python
authentication_classes = [JWTAuthentication]
permission_classes = [IsAuthenticated]
```

# Complete Example 1

```python
from rest_framework.permissions import IsAuthenticated
from rest_framework.viewsets import ModelViewSet
from .models import Note
from .serializers import NoteSerializer

class NoteViewSet(ModelViewSet):
    serializer_class = NoteSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Note.objects.filter(owner=self.request.user)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

Authentication must be configured globally or per-view. `IsAuthenticated` requires a user; queryset scoping authorizes which notes are visible; create assigns owner from trusted identity. Removing the scope leaves a cross-user object exposure even though callers are logged in.

# Complete Example 2

```python
from rest_framework.permissions import BasePermission, SAFE_METHODS

class ReadPublicWriteOwner(BasePermission):
    def has_object_permission(self, request, view, obj):
        if request.method in SAFE_METHODS and obj.is_public:
            return True
        return obj.owner_id == request.user.id
```

This class makes a per-object decision: safe reads of public objects pass; other access requires ownership. A production view also needs a suitable queryset and must ensure anonymous requests cannot dereference `request.user.id` unsafely; typically pair it with `IsAuthenticated` or explicitly handle anonymous users.

# Complete Example 3

```python
from rest_framework.test import APIClient

client = APIClient()
response = client.get("/api/notes/7/")
assert response.status_code in (401, 403, 404)
```

This is only a diagnostic placeholder: expected status depends on authentication scheme and whether the object is hidden by queryset scope. In a real test, create users and notes, authenticate as each user, and assert exact behavior.

# Advanced Example

Use a policy matrix to map roles and actions, then enforce it centrally with permission classes and scoped querysets. Avoid trusting role names or owner IDs supplied by the client.

# Under the Hood

DRF's permission checks execute after authentication. View permissions receive request/view; object permissions additionally receive the object, but are not automatically applied to every row of a list. List visibility must be enforced in `get_queryset()`.

# Security Notes

Return 404 for out-of-scope objects when existence should be concealed; return 403 when policy intentionally reveals existence. Never use either status as a substitute for correct access control.

# Best Practices

- Use authentication for identity and permissions for policy.
- Scope list and detail querysets.
- Set ownership and roles server-side.
- Test cross-user access, not only anonymous requests.

# Common Mistakes

- Treating `IsAuthenticated` as ownership enforcement.
- Expecting object permissions to filter list results.
- Checking a submitted `user_id` instead of `request.user`.

# Debugging

Trace authenticator result, permission classes, `get_queryset()`, object lookup, and object permission invocation. Compare list and detail behavior.

# Interview Questions

1. Difference between authentication and authorization?
2. Are object permissions applied to every object in a list?
3. Why use queryset scoping in addition to object permissions?

# Practice Exercises

1. Draw a role/action policy matrix for notes.
2. Implement private note access and cross-user tests.
3. Explain when returning 404 rather than 403 is appropriate.

# Chapter Summary

Authentication answers who; authorization answers what they may do. Secure DRF APIs combine identity, view/object policies, and queryset-level visibility.

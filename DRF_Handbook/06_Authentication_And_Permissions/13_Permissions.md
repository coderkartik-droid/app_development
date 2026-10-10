# Introduction

Permissions determine whether a request may proceed to a view and, where supported, whether a user may act on a particular object.

# Why this concept exists

Authentication identifies a caller, but applications need policies such as public reads, staff-only changes, or owner-only access.

# Real-world analogy

An employee badge identifies its holder; a room access list determines which doors it opens.

# Internal Working

DRF runs view-level permission checks during request initialization. Generic detail views call object-level checks after retrieval. List endpoints must filter their queryset because object checks are not automatically run for every returned row.

# Authentication Flow (where applicable)

Client -> authenticator -> request.user -> permission classes -> view/action -> serializer/model/database -> response.

# Syntax

```python
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
```

`IsAuthenticated` blocks anonymous requests. `get_queryset` ensures list and detail retrieval are limited to owned notes. Permission classes are evaluated before action logic; removing the queryset filter still exposes other users' records to authenticated callers.

# Complete Example 2

```python
from rest_framework.permissions import BasePermission, SAFE_METHODS

class IsOwnerOrReadOnly(BasePermission):
    def has_object_permission(self, request, view, obj):
        if request.method in SAFE_METHODS:
            return True
        return bool(request.user and request.user.is_authenticated
                    and obj.owner_id == request.user.pk)
```

The class allows safe methods and requires ownership for unsafe methods. It must be paired with suitable authentication and a view that invokes object checks. For private reads, also scope `get_queryset`; otherwise this permission allows every caller to read every object.

# Complete Example 3

```python
from rest_framework.test import APIRequestFactory, force_authenticate

factory = APIRequestFactory()
request = factory.get("/api/notes/")
force_authenticate(request, user=user)
```

This prepares an authenticated request for a focused permission/view test. `force_authenticate` skips credential parsing, so separately test the configured authentication mechanism in integration tests.

# Advanced Example

Compose permission policies carefully. DRF's permission list generally requires every listed permission to pass (AND); implement explicit composition or one class for OR policies. Document whether inaccessible objects are 403 or 404.

# Under the Hood

`APIView.check_permissions()` calls `has_permission`; generic object retrieval calls `check_object_permissions`, which invokes `has_object_permission`. A custom override that skips these hooks can bypass policy.

# Security Notes

Deny by default for private APIs, scope querysets, avoid trusting client-supplied roles, and test list/detail/create/update/delete for multiple identities.

# Best Practices

- Set global secure defaults.
- Use narrow view and object permissions.
- Enforce list visibility in queryset.
- Test both allowed and denied cases.

# Common Mistakes

- Confusing authentication with authorization.
- Expecting object permissions to filter list output.
- Forgetting custom object lookups must call permission checks.

# Debugging

Inspect `permission_classes`, authentication result, `has_permission`, queryset, object lookup, and `has_object_permission` invocation.

# Interview Questions

1. When are object permissions checked?
2. Why do list endpoints need queryset filtering?
3. How are multiple permission classes combined?

# Practice Exercises

1. Build owner-only CRUD with cross-user tests.
2. Implement public read/owner write.
3. Test a custom lookup for object permission enforcement.

# Chapter Summary

Permissions enforce policy at request and object boundaries. Secure list visibility with querysets and ensure every custom retrieval path runs the intended checks.

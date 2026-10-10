# Introduction

Custom permissions encode application-specific access rules that built-in classes cannot express clearly.

# Why this concept exists

Real policies involve ownership, membership, workflow state, role, or tenant relationships—not merely authenticated versus anonymous.

# Real-world analogy

A custom permission is a venue policy: a person may enter only if they belong to the event and have a ticket for the correct area.

# Internal Working

Implement `BasePermission.has_permission(request, view)` for request-level rules and optionally `has_object_permission(request, view, obj)` for object-specific rules.

# Authentication Flow (where applicable)

Credential -> request.user -> custom permission rule -> view/object -> serializer/database -> response.

# Syntax

```python
class CanManageProject(BasePermission):
    def has_object_permission(self, request, view, obj):
        return obj.members.filter(pk=request.user.pk).exists()
```

# Complete Example 1

```python
from rest_framework.permissions import BasePermission, SAFE_METHODS

class IsOwnerOrReadOnly(BasePermission):
    def has_object_permission(self, request, view, obj):
        if request.method in SAFE_METHODS:
            return True
        return bool(request.user and request.user.is_authenticated
                    and obj.owner_id == request.user.pk)
```

`SAFE_METHODS` contains GET, HEAD, and OPTIONS. Unsafe methods require an authenticated owner. This policy still permits public reads of every object when the queryset allows them; restrict the queryset for private resources. Add authentication at the view when the endpoint must always require login.

# Complete Example 2

```python
from rest_framework.permissions import BasePermission

class IsProjectMember(BasePermission):
    def has_permission(self, request, view):
        return bool(request.user and request.user.is_authenticated)

    def has_object_permission(self, request, view, obj):
        return obj.members.filter(pk=request.user.pk).exists()
```

The first method rejects anonymous requests before action logic; the second checks membership for a retrieved project. Add queryset scoping for list endpoints because object checks are not run for every list result. Index the membership relation for scale.

# Complete Example 3

```python
from rest_framework.test import APIClient

client = APIClient()
client.force_authenticate(user=owner)
response = client.patch(f"/api/projects/{project.pk}/", {"name": "New"}, format="json")
assert response.status_code == 200
```

The test bypasses authentication parsing but exercises the permission and view. Add a non-member test that asserts 403 or 404 according to the API's disclosure policy, plus anonymous and list tests.

# Advanced Example

Keep permission methods fast and side-effect free. If policy requires complex role resolution, prefetch or query efficiently, centralize policy logic, and ensure admin/background pathways use equivalent domain rules.

# Under the Hood

DRF instantiates permission classes per view request. A false result triggers `PermissionDenied` or authentication-related denial depending on authentication state and challenge behavior.

# Security Notes

Fail closed. Handle anonymous users explicitly; do not dereference `request.user` assuming it is authenticated. Avoid permission checks that trust request-body identity.

# Best Practices

- Separate request-level and object-level rules.
- Keep list scoping in `get_queryset`.
- Reuse named policy functions when rules span API/admin/services.
- Test all role/action/object combinations.

# Common Mistakes

- Implementing only `has_object_permission` while allowing anonymous access at the view level.
- Forgetting list scoping.
- Performing writes or external calls inside permission checks.

# Debugging

Set breakpoints/log safe identifiers in both methods; confirm the view invokes object checks and inspect the actual object/user IDs.

# Interview Questions

1. What are the two main BasePermission hooks?
2. Why is object permission insufficient for list filtering?
3. Should permission checks have side effects?

# Practice Exercises

1. Implement owner-or-read-only access.
2. Add project membership checks and list scoping.
3. Write a complete anonymous/member/non-member test matrix.

# Chapter Summary

Custom permissions express domain policy. Make them fail closed, efficient, side-effect free, and paired with queryset scope and comprehensive tests.

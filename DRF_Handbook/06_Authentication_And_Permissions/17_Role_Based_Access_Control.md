# Introduction

Role-based access control (RBAC) maps users to roles and roles to allowed capabilities, such as viewer, editor, or administrator.

# Why this concept exists

Checking individual user IDs throughout an application does not scale. Roles group policy while keeping access decisions understandable.

# Real-world analogy

At a hospital, a nurse, clinician, and billing clerk have different standard responsibilities. Their job role grants a set of capabilities, but patient-specific rules still apply.

# Internal Working

The API resolves the caller's trusted role or group membership, then evaluates whether that role may perform the action. Roles should come from server-controlled records/claims, not request payloads.

# Authentication Flow (where applicable)

Identity -> role/group lookup -> endpoint/object policy -> view -> serializer/model/database -> response.

# Syntax

```python
request.user.groups.filter(name="Editors").exists()
```

# Complete Example 1

```python
from rest_framework.permissions import BasePermission

class IsEditorOrAdmin(BasePermission):
    def has_permission(self, request, view):
        user = request.user
        return bool(
            user and user.is_authenticated and (
                user.is_superuser
                or user.groups.filter(name__in=["Editors", "Admins"]).exists()
            )
        )
```

The permission rejects anonymous users, grants superusers, and checks trusted Django group membership. Add this class to the relevant view and test every role. Groups are queried from server data; if called frequently, optimize carefully without stale authorization caches.

# Complete Example 2

```python
class ArticleViewSet(ModelViewSet):
    serializer_class = ArticleSerializer

    def get_permissions(self):
        if self.action in {"list", "retrieve"}:
            classes = [IsAuthenticated]
        else:
            classes = [IsEditorOrAdmin]
        return [permission() for permission in classes]
```

`get_permissions()` selects policy by ViewSet action. Public reads would use `AllowAny` deliberately; this example requires authentication for reads. Ensure custom actions are included in the correct policy branch, since an omitted action may accidentally receive a weaker default.

# Complete Example 3

```python
user.groups.add(editor_group)
assert user.groups.filter(name="Editors").exists()
```

This setup belongs in test fixtures or controlled administration, not an API field clients can set. Roles should be granted through audited workflows. Add a test proving a viewer cannot perform editor operations.

# Advanced Example

For complex systems, use capability permissions and tenant/record attributes alongside roles (RBAC plus ABAC). Avoid a single global `admin` switch when resource ownership, jurisdiction, or workflow state also matters.

# Under the Hood

Django groups aggregate permissions, but custom role-name checks do not automatically map to Django model permissions. Decide whether to use `user.has_perm()` or explicit group policy and keep the source of truth consistent.

# Security Notes

Role assignment is privileged. Audit grants/revocations, revoke stale sessions/claims where needed, and avoid trusting long-lived JWT role claims after role changes without a freshness strategy.

# Best Practices

- Define a role-capability matrix.
- Keep role assignment server-controlled and audited.
- Combine role checks with object/tenant scope.
- Test least-privilege behavior.

# Common Mistakes

- Trusting a `"role": "admin"` request field.
- Putting stale roles in long-lived tokens without revocation design.
- Treating role membership as sufficient for every object.

# Debugging

Inspect actual groups/permissions, action name, selected `get_permissions` branch, queryset scope, and role cache freshness.

# Interview Questions

1. What does RBAC centralize?
2. Why is request-provided role unsafe?
3. How does RBAC differ from object ownership?

# Practice Exercises

1. Create viewer/editor/admin capabilities.
2. Enforce role-specific custom action behavior.
3. Add tests for role revocation and tenant scope.

# Chapter Summary

RBAC makes broad capabilities manageable, but it must use trusted role assignments and be combined with per-object and tenant authorization.

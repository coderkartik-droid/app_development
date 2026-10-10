# Introduction

Object-level permissions decide whether a caller may perform an operation on one specific object, such as editing one order or viewing one document.

# Why this concept exists

View-level policy can say “authenticated users may use this endpoint,” but individual records may belong to different owners or teams.

# Real-world analogy

A building visitor may enter the office, but a key for one cabinet does not grant access to every cabinet.

# Internal Working

Generic detail views retrieve an object and invoke `check_object_permissions(request, obj)`. Lists do not invoke this for each row; constrain the list queryset separately.

# Authentication Flow (where applicable)

Authenticate -> view permission -> scoped lookup -> object permission -> handler -> serializer/database -> response.

# Syntax

```python
def has_object_permission(self, request, view, obj):
    return obj.owner_id == request.user.pk
```

# Complete Example 1

```python
from rest_framework.permissions import BasePermission, IsAuthenticated
from rest_framework.viewsets import ModelViewSet

class IsDocumentOwner(BasePermission):
    def has_object_permission(self, request, view, obj):
        return obj.owner_id == request.user.pk

class DocumentViewSet(ModelViewSet):
    serializer_class = DocumentSerializer
    permission_classes = [IsAuthenticated, IsDocumentOwner]

    def get_queryset(self):
        return Document.objects.filter(owner=self.request.user)
```

Authentication ensures `request.user` is real. The permission checks a detail object; queryset scope limits list and lookup. The duplicate owner rule is intentional defense-in-depth and can produce 404 for out-of-scope objects before object permission runs. Ensure `owner_id` is not client-writable.

# Complete Example 2

```python
from rest_framework.generics import RetrieveUpdateAPIView

class DocumentDetailView(RetrieveUpdateAPIView):
    serializer_class = DocumentSerializer
    permission_classes = [IsAuthenticated, IsDocumentOwner]

    def get_queryset(self):
        return Document.objects.filter(team__members=self.request.user)
```

This allows team-member retrieval candidates; the object permission can impose stricter ownership for updates, but as written it also applies to GET and would reject non-owners. If reads and writes differ, encode method-aware rules explicitly. Queryset scoping still protects cross-team records.

# Complete Example 3

```python
response = APIClient().get(f"/api/documents/{other_user_document.pk}/")
assert response.status_code in (403, 404)
```

Choose and assert one documented behavior. A scoped queryset generally returns 404; an object permission after broad retrieval may return 403. Test both list exclusion and detail access; one does not prove the other.

# Advanced Example

For nested resources, ensure the child belongs to the parent in the URL. A correct child permission cannot compensate for a route that retrieves a child globally and ignores the parent identifier.

# Under the Hood

`GenericAPIView.get_object()` applies lookup to `get_queryset()`, then calls `check_object_permissions`. Custom views that fetch objects directly must call the check themselves or use the generic helper.

# Security Notes

Prevent insecure direct object references by enforcing authorization on every lookup. Opaque UUIDs reduce casual guessing but are not authorization.

# Best Practices

- Combine object permissions with queryset scoping.
- Make read/write rules explicit.
- Test cross-user and cross-tenant list/detail paths.
- Use trusted ownership fields.

# Common Mistakes

- Expecting object rules to filter lists.
- Retrieving with unrestricted `Model.objects.get`.
- Assuming hidden IDs are secure.

# Debugging

Trace lookup queryset, `get_object`, permission methods, method-specific branches, and parent-child constraints.

# Interview Questions

1. When does DRF invoke object permissions?
2. Why must list querysets be scoped?
3. What is an insecure direct object reference?

# Practice Exercises

1. Secure document detail and list for owner only.
2. Allow team reads but owner-only updates.
3. Test nested parent-child access.

# Chapter Summary

Object permissions handle per-record authorization, but only when the view invokes them. Secure lists and details with both scoped querysets and correct object checks.

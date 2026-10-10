# Introduction

`update(self, instance, validated_data)` defines how `serializer.save()` changes an existing object. It is invoked when the serializer was initialized with an `instance`.

# Why this concept exists

An update may need to protect immutable fields, handle optional values, or update related records transactionally. Explicit update behavior prevents accidental replacement of fields or partial database changes.

# Real-world analogy

Updating a record is like editing a signed form: preserve fields the editor did not change, verify changed values, and record the approved result.

# Internal Working

```text
instance + request.data + partial flag -> validation -> save()
 -> update(instance, validated_data) -> persist -> representation
```

For PUT, fields are generally expected; for PATCH, `partial=True` allows omitted fields. Omitted fields and explicitly provided null are not equivalent.

# Request Lifecycle (where applicable)

The view retrieves an authorized object, binds it and request data to the serializer, validates, then saves. Queryset scoping must happen before serialization so a caller cannot update another tenant's object.

# Syntax

```python
def update(self, instance, validated_data):
    instance.name = validated_data.get("name", instance.name)
    instance.save()
    return instance
```

# Complete Example 1

Assume a `Profile` model has `display_name` and `bio`; this serializer updates only submitted fields.

```python
# profiles/serializers.py
from rest_framework import serializers
from .models import Profile


class ProfileSerializer(serializers.ModelSerializer):
    class Meta:
        model = Profile
        fields = ["id", "display_name", "bio"]
        read_only_fields = ["id"]

    def update(self, instance, validated_data):
        instance.display_name = validated_data.get(
            "display_name", instance.display_name
        )
        instance.bio = validated_data.get("bio", instance.bio)
        instance.save(update_fields=["display_name", "bio"])
        return instance
```

```python
# profiles/views.py
from rest_framework import generics
from rest_framework.permissions import IsAuthenticated
from .models import Profile
from .serializers import ProfileSerializer


class ProfileDetailView(generics.RetrieveUpdateAPIView):
    serializer_class = ProfileSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Profile.objects.filter(user=self.request.user)
```

```python
# profiles/urls.py
from django.urls import path
from .views import ProfileDetailView

urlpatterns = [path("profile/<int:pk>/", ProfileDetailView.as_view())]
```

**Line-by-line:** Meta allowlists fields. `update` uses `.get` so omitted values remain unchanged, persists the instance, and returns it. The generic detail view retrieves, updates, and serializes; `get_queryset` restricts records to the authenticated user. URL includes a primary key.

**Request → Serializer → Validation → Database → Response:** route -> owner-scoped queryset lookup -> serializer binds instance/data -> fields validate -> update saves columns -> output representation -> response. **Internal working:** With `partial=True`, only supplied fields are in validated data. Removing queryset scoping can create an insecure object-level access flaw; removing return breaks serializer's update contract.

**Professional use case:** Profile editing where the user may edit only their own record.

# Complete Example 2

A generic ModelSerializer often needs no custom update:

```python
serializer = ProfileSerializer(instance=profile, data=request.data, partial=True)
serializer.is_valid(raise_exception=True)
profile = serializer.save()
```

Default update assigns validated fields and saves. Prefer this unless custom rules are needed. The view still has to retrieve an authorized instance.

# Complete Example 3

Wrap related changes atomically:

```python
from django.db import transaction


def update(self, instance, validated_data):
    with transaction.atomic():
        instance.name = validated_data.get("name", instance.name)
        instance.save(update_fields=["name"])
        AuditEvent.objects.create(target=instance, action="updated")
    return instance
```

If audit creation fails, the update rolls back. Do not send external messages while holding the transaction; use an after-commit mechanism when needed.

# Advanced Example

Writable nested updates require deciding whether omitted children are retained, removed, or ignored, and how matching identities are determined. Implement a deliberate algorithm and transaction; do not rely on default nested serializer behavior.

# Under the Hood

`save()` calls `update(instance, validated_data)` when `instance` is present. The returned object becomes serializer `.instance`, which is then represented through `.data`.

# Performance Notes

Use `update_fields` when appropriate, but account for auto-updated fields and model save hooks. Avoid loading relations repeatedly in update workflows.

# Best Practices

- Retrieve instances from an authorized queryset.
- Distinguish PUT and PATCH behavior.
- Preserve omitted fields intentionally.
- Return the updated instance and use transactions for related writes.

# Common Mistakes

- Assuming `partial=True` authorizes the target object.
- Replacing omitted fields with `None`.
- Forgetting to return the instance.
- Implementing nested update without identity/delete rules.

# Debugging

Inspect `instance`, `initial_data`, `validated_data`, `partial`, and changed database fields. Test omitted vs null input and unauthorized IDs.

# Interview Questions

1. When does `update()` run? `save()` with an existing instance.
2. What changes for PATCH? Omitted fields can be allowed via `partial=True`.
3. What protects object ownership? Authorized queryset/permission logic.
4. Why transaction related updates? Avoid partial persistence.

# Practice Exercises

1. Add a PATCH endpoint that changes only supplied profile fields.
2. Test omission, explicit null, and invalid field values.
3. Test that a user cannot update another user's profile.

# Chapter Summary

Update receives an existing instance and validated changes. Scope object lookup securely, preserve omitted fields, distinguish null from omission, and make related persistence atomic.

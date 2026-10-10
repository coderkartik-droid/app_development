# Introduction

`create(self, validated_data)` defines how a serializer creates a new object when `save()` is called without an existing `instance`. ModelSerializer supplies a default for simple model creation; override it for domain-specific creation behavior.

# Why this concept exists

Real creation often includes server-owned fields, normalization, related records, or a required transaction. A custom method makes those rules explicit rather than accepting client control over them.

# Real-world analogy

The create method is an intake clerk who turns an approved application into an official record and fills in internal fields the applicant cannot choose.

# Internal Working

```text
Serializer(data=...) -> is_valid() -> save(**extra)
 -> create({**validated_data, **extra}) -> model/service -> instance
```

`save()` requires successful validation. Extra keyword arguments passed to `save()` are merged into validated data for persistence but are not validated as client fields.

# Request Lifecycle (where applicable)

The API view parses the request, serializer validates it, and `serializer.save(owner=request.user)` invokes `create`. The result instance is serialized into the response.

# Syntax

```python
def create(self, validated_data):
    return Model.objects.create(**validated_data)
```

# Complete Example 1

Assume a `Task` model has `title` and `owner` fields. The client may provide only a title; the authenticated user owns the created task.

```python
# tasks/serializers.py
from rest_framework import serializers
from .models import Task


class TaskCreateSerializer(serializers.ModelSerializer):
    class Meta:
        model = Task
        fields = ["id", "title"]
        read_only_fields = ["id"]

    def create(self, validated_data):
        owner = self.context["request"].user
        return Task.objects.create(owner=owner, **validated_data)
```

```python
# tasks/views.py
from rest_framework import generics
from rest_framework.permissions import IsAuthenticated
from .serializers import TaskCreateSerializer


class TaskCreateView(generics.CreateAPIView):
    serializer_class = TaskCreateSerializer
    permission_classes = [IsAuthenticated]
```

```python
# tasks/urls.py
from django.urls import path
from .views import TaskCreateView

urlpatterns = [path("tasks/", TaskCreateView.as_view(), name="task-create")]
```

**Line-by-line:** Meta maps the serializer to `Task`, exposes only ID/title, and keeps ID read-only. `create` obtains request from serializer context and creates the row with server-selected owner. `CreateAPIView` supplies POST behavior and serializer context automatically. `IsAuthenticated` ensures a user exists. URL binds the collection path.

**Request → Serializer → Validation → Database → Response:** JSON is parsed -> title validated -> generic view saves -> custom create assigns authenticated owner and ORM inserts -> serializer represents task -> response. **Internal working:** The client cannot set `owner` because it is absent from writable fields. Removing authentication can make context user anonymous; removing server ownership can create unowned or spoofed tasks.

**Professional use case:** Tenant-scoped records where ownership must come from authenticated identity.

# Complete Example 2

Use `serializer.save(owner=request.user)` with a simpler ModelSerializer default `create`, or override `perform_create` in a generic view. This is often preferable when only injecting a server value; override `create()` for deeper persistence behavior.

```python
def perform_create(self, serializer):
    serializer.save(owner=self.request.user)
```

The generic view calls this after validation. The owner is an extra save value, not a client-submitted validated field. Ensure permission/authentication policy exists.

# Complete Example 3

Create related records atomically:

```python
from django.db import transaction


def create(self, validated_data):
    with transaction.atomic():
        parent = Parent.objects.create(**validated_data)
        Child.objects.create(parent=parent, label="initial")
    return parent
```

If child creation fails, the parent insert rolls back. Keep external network operations outside the database transaction.

# Advanced Example

For complex workflows, call a domain service from `create()` or the view. Serializer creation should not become an unbounded workflow engine; make transaction ownership and side effects explicit.

# Under the Hood

`BaseSerializer.save()` checks that validation has run and succeeded, then calls `create(validated_data)` when `instance` is absent. It stores the returned instance and makes it available through `.instance` and `.data`.

# Performance Notes

Creation usually performs one insert, but nested relations/validators may add queries. Use bulk operations only when semantics, signals, and validation requirements are understood.

# Best Practices

- Validate before save.
- Derive ownership and trusted fields server-side.
- Use transactions for related writes.
- Return the created instance.

# Common Mistakes

- Forgetting to return the object.
- Calling `save()` before `is_valid()`.
- Trusting client-provided ownership.
- Overriding create for simple behavior already handled by `perform_create`.

# Debugging

Inspect `validated_data`, serializer context, transaction errors, and the returned instance. Confirm the model manager is appropriate for special creation rules.

# Interview Questions

1. When does `create()` run? `save()` with no existing instance.
2. What does it receive? Validated values, plus extra `save()` kwargs merged in.
3. Where assign owner? From trusted request context/server logic.
4. Why return the instance? DRF uses it for serializer state and output representation.

# Practice Exercises

1. Create a task whose owner is the authenticated user.
2. Add a related event within a transaction.
3. Test that client-supplied owner IDs are ignored or rejected.

# Chapter Summary

Override `create()` when persistence needs custom behavior; otherwise prefer generic defaults or `perform_create`. Always save validated data, assign trusted fields server-side, and return the created instance.

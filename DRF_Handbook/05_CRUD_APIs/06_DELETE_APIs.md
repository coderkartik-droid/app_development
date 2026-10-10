# Introduction

`DELETE` asks the server to remove a resource or make it unavailable. The response is often `204 No Content`; repeated deletion requests should not cause unexpected side effects.

# Why this concept exists

Applications need a standard way to retire records and associated behavior while preserving integrity and audit requirements.

# Real-world analogy

Deleting a catalogue entry is like withdrawing a product card from public shelves. The business may physically erase it, archive it, or retain it for accounting.

# Internal Working

DRF resolves the detail object using a queryset, checks view/object permissions, calls the destroy hook, and returns a response. Django's model deletion can cascade to related rows according to relationship configuration.

# Request Lifecycle

`DELETE /products/42/` -> detail route -> destroy -> scoped `get_object()` -> permissions -> delete/archive -> 204 response.

# Syntax

```http
DELETE /api/products/42/
Authorization: Bearer <token>
```

# Complete Example 1

```python
from rest_framework import generics
from rest_framework.permissions import IsAdminUser
from .models import Product
from .serializers import ProductSerializer

class ProductDeleteView(generics.DestroyAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAdminUser]
```

`DestroyAPIView` supplies DELETE behavior; `IsAdminUser` restricts it to staff. `get_object()` still uses the queryset and object permission hooks. Production apps often scope by tenant and require stronger policy than staff status. Removing the queryset or permission can make unintended records deletable.

# Complete Example 2

Soft-delete by marking inactive:

```python
from rest_framework import status, viewsets
from rest_framework.response import Response

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer

    def perform_destroy(self, instance):
        instance.active = False
        instance.save(update_fields=["active"])
```

`perform_destroy` is called by the standard destroy action, so the response remains the framework's normal 204. This code assumes `active` means withdrawn; add audit metadata and hide inactive rows in public list/detail querysets if that is the policy. Soft delete does not automatically protect related data.

# Complete Example 3

```python
from rest_framework.test import APIClient

client = APIClient()
response = client.delete("/api/products/42/")
assert response.status_code == 204
assert response.content == b""
```

This verifies the no-content contract. Create a test product first and confirm it is deleted or archived according to the application policy. Also test unauthorized and missing-object behavior.

# Advanced Example

For irreversible or regulated data removal, implement authorization, confirmation policy, audit trail, and transactional handling of related records. Consider asynchronous cleanup for large external resources; do not block a request on unbounded cascading work.

# Under the Hood

`DestroyModelMixin.destroy()` gets the object, calls `perform_destroy(instance)`, then returns `Response(status=204)`. `Model.delete()` issues SQL and may invoke cascades/signals. Database constraints govern protected relationships.

# Performance Notes

Large cascades can be costly. Profile delete queries and use bounded background cleanup where required. Add indexes on foreign keys that participate in cascades.

# Best Practices

- Decide hard-delete versus archive explicitly.
- Restrict deletion to authorized actors.
- Preserve audit requirements and foreign-key integrity.
- Return 204 without a response body when following the common contract.

# Common Mistakes

- Assuming hidden UI controls provide authorization.
- Soft-deleting without filtering archived rows from reads.
- Deleting parent records without understanding cascade behavior.

# Debugging

Inspect permission failures, queryset scope, model `on_delete`, signals, transaction logs, and response status. Verify the data's intended post-delete state.

# Interview Questions

1. What does a 204 response mean? Success with no response representation.
2. What is soft deletion? Marking a row unavailable instead of physically removing it.
3. Does Django cascade deletion by default? Relationship `on_delete` behavior determines it.

# Practice Exercises

1. Restrict deletion to an object's owner.
2. Implement and test soft-delete filtering.
3. Draw the related rows affected by a hard delete.

# Chapter Summary

DELETE semantics are a business and integrity decision. DRF provides a destroy path, but access control, cascade behavior, auditability, and archival visibility are application responsibilities.

# Introduction

`POST` commonly creates a resource or triggers a collection-level operation. For resource creation, clients submit input; DRF parses it, validates it through a serializer, and persists an instance.

# Why this concept exists

POST gives clients a standard way to request creation without choosing database identifiers or writing directly to storage.

# Real-world analogy

Submitting a product intake form asks the warehouse to create a catalogue record. The warehouse checks the form and supplies internal details such as its database ID.

# Internal Working

The request body parser produces primitive data. Serializer fields coerce and validate values; `is_valid()` builds `validated_data`; `save()` calls `create()`. DRF then serializes the created object and normally returns 201.

# Request Lifecycle

`POST /products/` -> router -> create handler -> permission -> parser -> serializer validation -> `perform_create`/save -> INSERT -> response serializer -> 201 Created.

# Syntax

```http
POST /api/products/
Content-Type: application/json

{"name":"Desk lamp","sku":"LAMP-01","price":"24.90","active":true}
```

# Complete Example 1

```python
# serializers.py
from rest_framework import serializers
from .models import Product

class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ["id", "name", "sku", "price", "active"]
        read_only_fields = ["id"]
```

```python
# views.py
from rest_framework import generics
from rest_framework.permissions import IsAuthenticated
from .models import Product
from .serializers import ProductSerializer

class ProductCreateView(generics.CreateAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticated]
```

`CreateAPIView` provides POST creation. The serializer exposes writable fields and keeps the server-generated ID read-only. The permission requires a signed-in user; a production app may also need role authorization and throttling. Removing validation is unsafe; clients can submit malformed values or a duplicate SKU.

# Complete Example 2

```python
# In a create view for a model with an owner foreign key:
def perform_create(self, serializer):
    serializer.save(owner=self.request.user)
```

The hook adds a trusted server-side value rather than trusting `owner` from the body. The serializer/model must define that relation. If this hook is removed, owner may be missing or client-controlled depending on serializer configuration. Reject anonymous requests before reaching it.

# Complete Example 3

```python
from rest_framework.test import APIClient

client = APIClient()
response = client.post(
    "/api/products/",
    {"name": "Desk lamp", "sku": "LAMP-01", "price": "24.90", "active": True},
    format="json",
)
assert response.status_code == 201, response.data
assert response.data["sku"] == "LAMP-01"
```

The test uses JSON encoding and verifies both status and representation. Add authentication and database cleanup/fixtures in the test suite. A duplicate SKU should be tested as a validation error.

# Advanced Example

For creation involving several dependent rows, validate first and write inside `transaction.atomic()`. Use an idempotency strategy for operations where retries could create duplicates; POST is not inherently idempotent.

# Under the Hood

`CreateModelMixin.create()` instantiates the serializer with `request.data`, validates, calls `perform_create`, and returns `Response(serializer.data, status=201)`. Validation errors become a structured 400 response.

# Performance Notes

Keep create validation bounded; avoid one query per submitted row. Use bulk creation only when its validation and signal semantics are deliberately handled.

# Best Practices

- Set trusted ownership/tenant fields on the server.
- Return 201 and, where useful, a `Location` header.
- Enforce uniqueness and business rules; handle concurrent conflicts.
- Document required fields and error format.

# Common Mistakes

- Calling `.save()` before `.is_valid()`.
- Accepting admin-only fields from user input.
- Assuming `POST` retries are safe.

# Debugging

Inspect `request.content_type`, parser configuration, `serializer.errors`, database constraints, and whether `perform_create` runs. Use a test with malformed and duplicate values.

# Interview Questions

1. Why does POST usually return 201? A new resource was created.
2. What is `perform_create` for? Customizing save-time trusted values.
3. Is POST idempotent? Not by default.

# Practice Exercises

1. Create a product API that assigns the current staff user as creator.
2. Validate SKU format and uniqueness.
3. Test valid, invalid, and unauthorized create requests.

# Chapter Summary

POST creation is a validated boundary: parse, validate, enforce policy, save trusted data, and return a clear representation. Database constraints remain necessary for integrity under concurrent requests.

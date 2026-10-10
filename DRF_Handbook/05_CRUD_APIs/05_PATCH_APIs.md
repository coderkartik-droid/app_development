# Introduction

`PATCH` changes only specified fields. It is appropriate for forms that edit one value, status transitions, and clients that do not hold a complete current representation.

# Why this concept exists

Clients often update one small part of a record. Requiring every field creates brittle clients and can overwrite values the client does not know about.

# Real-world analogy

PATCH is a correction slip saying “change the price to 19.95”; all other product details remain as they were.

# Internal Working

DRF's update mixin invokes `partial_update`, sets `partial=True`, retrieves the authorized instance, and validates only supplied fields (while still running applicable object-level validation).

# Request Lifecycle

`PATCH /products/42/` -> detail action -> scoped object lookup -> serializer(partial=True) -> validate supplied values -> UPDATE provided fields -> response.

# Syntax

```http
PATCH /api/products/42/
Content-Type: application/json

{"price":"19.95"}
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

class ProductPatchView(generics.UpdateAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticated]
```

The generic update view handles PATCH by using partial serializer validation. The queryset must be narrowed to the caller's permitted records. Removing instance scoping can expose writes across users; making fields read-only prevents clients from changing them.

# Complete Example 2

```python
product = Product.objects.get(pk=42)
serializer = ProductSerializer(product, data={"active": False}, partial=True)
serializer.is_valid(raise_exception=True)
serializer.save()
```

The instance selects update mode, `partial=True` permits omitted fields, and `save()` invokes serializer update. A direct database lookup is shown to explain the API; production view code should retrieve through the request-scoped queryset and perform object permission checks.

# Complete Example 3

```python
from rest_framework.test import APIClient

client = APIClient()
response = client.patch("/api/products/42/", {"price": "19.95"}, format="json")
assert response.status_code == 200
assert response.data["price"] == "19.95"
```

The test sends only the changed field and confirms the returned representation. Create the fixture with a known original name and assert it remains unchanged to verify partial semantics.

# Advanced Example

For state changes with business rules (e.g. order `pending` to `shipped`), consider a dedicated action instead of allowing arbitrary PATCH to change status. Validate allowed transitions and record actor/audit time atomically.

# Under the Hood

`partial_update()` delegates to `update()` with a partial flag. Serializer fields omitted from the input are not overwritten; supplied fields are validated and passed to `update()`.

# Performance Notes

PATCH reduces payload size but does not automatically reduce ORM work. Use atomic updates carefully only when serializer validation and model hooks remain correctly applied.

# Best Practices

- Limit fields clients may patch.
- Validate cross-field invariants using the instance plus incoming attributes.
- Return a clear updated representation or consistently use 204.
- Audit sensitive changes.

# Common Mistakes

- Assuming partial updates bypass validation.
- Letting arbitrary clients patch workflow status or ownership.
- Treating JSON Merge Patch semantics as automatic; content-type and semantics must be designed.

# Debugging

Check method dispatch, `partial=True`, submitted field names, read-only configuration, object lookup, and serializer errors.

# Interview Questions

1. Why use PATCH? To express a partial modification.
2. Are supplied PATCH values validated? Yes.
3. Why restrict patchable fields? To preserve invariants and server-owned data.

# Practice Exercises

1. Add a PATCH endpoint that permits description edits only.
2. Test that omitted values are preserved.
3. Implement a validated status transition endpoint.

# Chapter Summary

PATCH updates only supplied fields, but still requires authorization, validation, and integrity rules. For important workflow transitions, prefer a named domain operation over unconstrained field updates.

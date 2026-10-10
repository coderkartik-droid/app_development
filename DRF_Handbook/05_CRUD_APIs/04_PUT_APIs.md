# Introduction

`PUT` updates a resource using replacement semantics: the client supplies the complete writable representation expected by the endpoint. Missing required fields should generally fail validation rather than silently retain old values.

# Why this concept exists

Replacement semantics make a client's intended final state explicit and allow repeated identical requests to converge on the same state.

# Real-world analogy

PUT is submitting a complete replacement product specification sheet, not a sticky note containing one correction.

# Internal Working

DRF retrieves the target object through the view's queryset, checks object-level permissions, constructs the serializer with both instance and request data, validates required fields, and calls `update()` on save.

# Request Lifecycle

`PUT /products/42/` -> detail route -> update action -> scoped object lookup -> permission -> serializer with `partial=False` -> UPDATE -> serialized response.

# Syntax

```http
PUT /api/products/42/
Content-Type: application/json

{"name":"Desk lamp","sku":"LAMP-01","price":"19.95","active":true}
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

class ProductUpdateView(generics.UpdateAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticated]
```

`UpdateAPIView` maps PUT to full update and PATCH to partial update. The serializer describes writable fields. Scope `queryset` by tenant/owner if records are private. If the serializer has required fields, omitting them in PUT yields validation errors, which is the intended distinction from PATCH.

# Complete Example 2

```python
product = Product.objects.get(pk=42)
serializer = ProductSerializer(
    product,
    data={"name": "Desk lamp", "sku": "LAMP-01", "price": "19.95", "active": True},
)
serializer.is_valid(raise_exception=True)
updated = serializer.save()
```

Providing `product` as the first positional argument selects update behavior; omitting `partial=True` preserves replacement semantics. `raise_exception=True` lets DRF convert validation errors to its standard response. In an endpoint, do not use an unrestricted `.get()`; use `get_object()` or an equivalent scoped lookup.

# Complete Example 3

```python
from rest_framework.test import APIClient

client = APIClient()
response = client.put(
    "/api/products/42/",
    {"name": "Desk lamp", "sku": "LAMP-01", "price": "19.95", "active": True},
    format="json",
)
assert response.status_code in (200, 404)
```

This illustrates the request shape, but a real test should create product 42 and assert exactly 200 plus updated fields. The 404 alternative is useful only in a separate not-found test. Keep tests explicit to catch routing regressions.

# Advanced Example

When replacement includes nested objects, define whether omitted nested data is rejected, cleared, or left unchanged. DRF does not infer a safe nested replacement contract; implement and test it explicitly.

# Under the Hood

`UpdateModelMixin.update()` calls `get_object()`, creates a serializer with the object and `request.data`, validates, runs `perform_update`, then returns serialized data. `partial_update()` sets `partial=True`.

# Performance Notes

Use scoped indexed lookups and avoid fetching expensive unrelated relationships. Keep validation queries bounded and beware uniqueness checks under concurrent updates.

# Best Practices

- Document complete field requirements for PUT.
- Exclude immutable/internal fields from writable fields.
- Use object permissions and scoped querysets.
- Test omitted required fields and nonexistent IDs.

# Common Mistakes

- Treating PUT as PATCH.
- Replacing trusted owner/creation metadata from client input.
- Returning stale data by serializing a different instance.

# Debugging

Check method mapping, `partial` setting, required field errors, target queryset scope, and serializer `update()` behavior. Confirm the DB row after response.

# Interview Questions

1. How does PUT differ from PATCH? Full replacement intent versus partial changes.
2. What does `partial=True` change? Missing required fields are permitted.
3. What is the risk of unrestricted object retrieval? Unauthorized object modification.

# Practice Exercises

1. Test a PUT with one required field missing.
2. Make a product's SKU immutable after creation.
3. Explain how nested replacement should behave in your API.

# Chapter Summary

PUT communicates full replacement. DRF retrieves a permitted object, validates complete writable input, persists through serializer update, and returns the representation. Use PATCH when only selected fields should change.

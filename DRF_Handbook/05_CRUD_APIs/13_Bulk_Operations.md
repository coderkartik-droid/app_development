# Introduction

Bulk operations create or update multiple records in one request. They reduce network round trips but introduce atomicity, validation, partial-failure, and resource-limit decisions.

# Why this concept exists

Import tools and administrative clients may need to process many records more efficiently than sending one request per row.

# Real-world analogy

Bulk import is submitting a batch inventory sheet instead of filling a separate form for every product.

# Internal Working

The API validates a bounded list, checks per-item and cross-item constraints, then performs writes in a transaction or reports item-level outcomes according to a defined contract.

# Request Lifecycle

POST batch -> parser -> list serializer -> validate each entry and batch invariants -> transaction/ORM bulk operation -> response with created records or indexed errors.

# Syntax

```python
items = ProductSerializer(data=request.data, many=True)
items.is_valid(raise_exception=True)
```

# Complete Example 1

```python
from rest_framework import generics, serializers
from rest_framework.response import Response
from rest_framework import status
from django.db import transaction
from .models import Product

class ProductInputSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ["name", "sku", "price", "active"]

class ProductBulkCreateView(generics.GenericAPIView):
    serializer_class = ProductInputSerializer

    def post(self, request):
        serializer = self.get_serializer(data=request.data, many=True)
        serializer.is_valid(raise_exception=True)
        with transaction.atomic():
            products = serializer.save()
        return Response(ProductInputSerializer(products, many=True).data,
                        status=status.HTTP_201_CREATED)
```

`many=True` creates a list serializer; validation reports item errors. `transaction.atomic()` commits all created rows or rolls them all back if an exception occurs. Add a maximum batch length before expensive validation, and decide how duplicate SKUs within the same batch are reported. Bulk `save()` delegates to list serializer behavior.

# Complete Example 2

```python
from rest_framework.exceptions import ValidationError

class ProductInputSerializer(serializers.ModelSerializer):
    def validate(self, attrs):
        sku = attrs.get("sku")
        if sku and Product.objects.filter(sku=sku).exists():
            raise ValidationError({"sku": "Already exists."})
        return attrs
```

This field/object validation illustrates an existing-record check but by itself may query once per item and can race. Production should use a database uniqueness constraint and translate `IntegrityError` safely; batch-level duplicate detection should also be explicit.

# Complete Example 3

```python
response = APIClient().post("/api/products/bulk/",
    [{"name": "A", "sku": "A-1", "price": "2.00", "active": True}],
    format="json")
assert response.status_code == 201
```

The test sends an array rather than an object and confirms creation status. Also test empty batches, malformed rows, excessive length, and rollback after a failing row.

# Advanced Example

If partial success is needed, return a documented per-item result with stable client identifiers and errors. Do not accidentally promise all-or-nothing semantics while committing each row independently.

# Under the Hood

DRF's `ListSerializer` coordinates child serializer validation and save. ORM `bulk_create` can reduce SQL but may bypass model `save()` methods/signals and does not behave identically across database backends.

# Performance Notes

Cap batch count and payload bytes, chunk very large imports, measure query counts, and avoid N+1 validation queries.

# Best Practices

- Decide atomic versus partial success up front.
- Enforce batch size limits and database constraints.
- Return actionable per-row errors without leaking internals.
- Use `bulk_create` only after understanding skipped hooks and backend behavior.

# Common Mistakes

- Unbounded batches.
- Assuming `bulk_create` runs every model hook.
- Ignoring duplicate values within the submitted batch.

# Debugging

Inspect list serializer errors by index, transaction rollback, database constraints, and batch limits. Test failure in the middle of a batch.

# Interview Questions

1. What does `many=True` do? Uses a ListSerializer to process multiple items.
2. What does `atomic()` guarantee? All transaction writes commit or roll back together.
3. What may `bulk_create` bypass? Per-instance save behavior and signals.

# Practice Exercises

1. Implement atomic bulk product creation with a maximum of 100 rows.
2. Add duplicate-SKU detection within one payload.
3. Specify a partial-success response contract.

# Chapter Summary

Bulk APIs trade fewer round trips for more complex semantics. Bound input, validate batch rules, choose transaction behavior, and understand ORM bulk-operation limitations.

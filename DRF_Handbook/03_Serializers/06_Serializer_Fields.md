# Introduction

Serializer fields define the names, types, conversion rules, validation, and output representation of values in a DRF contract. A field can be inferred from a model or declared explicitly.

# Why this concept exists

JSON values are loosely typed, while application code needs predictable Python values and validation. Fields provide one consistent conversion boundary.

# Real-world analogy

Each serializer field is a labeled box on a form with instructions about what can go inside and how it should be written on the receipt.

# Internal Working

During binding, DRF associates each field with its serializer and name. Input fields convert primitive values with `to_internal_value`, validate, and add values to `validated_data`; output fields use `to_representation`.

# Request Lifecycle (where applicable)

```text
request.data -> field conversion -> field validators -> object validation
             -> validated_data -> create/update -> field representation -> Response
```

# Syntax

```python
name = serializers.CharField(max_length=120)
price = serializers.DecimalField(max_digits=8, decimal_places=2)
created_at = serializers.DateTimeField(read_only=True)
```

# Complete Example 1

This standalone serializer validates a checkout line command. It does not directly save a database record.

```python
# api/serializers.py
from rest_framework import serializers


class CheckoutLineSerializer(serializers.Serializer):
    product_id = serializers.IntegerField(min_value=1)
    quantity = serializers.IntegerField(min_value=1, max_value=50)
    note = serializers.CharField(
        max_length=200,
        required=False,
        allow_blank=True,
        default="",
    )
```

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .serializers import CheckoutLineSerializer


@api_view(["POST"])
def validate_checkout_line(request):
    serializer = CheckoutLineSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    return Response({"validated": serializer.validated_data})
```

```python
# api/urls.py
from django.urls import path
from .views import validate_checkout_line

urlpatterns = [path("checkout-line/", validate_checkout_line)]
```

**Line-by-line:** `IntegerField` parses integer-like input and enforces a positive product key; quantity has lower and upper bounds. `CharField` limits note length, makes it optional, allows empty text, and supplies an empty default. The view binds request data, validates, then returns typed `validated_data`. The URL exposes the view.

**Request → Serializer → Validation → Database → Response:** parser creates primitives -> fields coerce/check values -> no database is used in this validation-only example -> response renderer returns validated values. **Internal working:** A field's `required`, `allow_null`, `allow_blank`, and `default` are distinct options. Removing `max_value` permits unreasonable quantity input; removing `is_valid` bypasses all field rules.

**Professional use case:** Validate an operation payload before a service checks product availability and performs a transactional order.

# Complete Example 2

Use `EmailField` for a formatted email:

```python
class InviteSerializer(serializers.Serializer):
    email = serializers.EmailField()
```

It parses a string and applies email-format validation; it does not prove the address exists or belongs to the caller. A client-provided email still requires verification for identity-sensitive workflows.

# Complete Example 3

Use a relation field in a model serializer:

```python
class OrderSerializer(serializers.ModelSerializer):
    customer = serializers.PrimaryKeyRelatedField(read_only=True)

    class Meta:
        model = Order
        fields = ["id", "customer", "created_at"]
```

The representation uses a related object's primary key. `read_only=True` prevents client reassignment through this field. Related lookup/query cost should be considered when serializing collections.

# Advanced Example

Select field classes based on storage and API contract: `UUIDField`, `DateField`, `DateTimeField`, `ChoiceField`, `ListField`, `DictField`, and related fields each provide conversion and validation. Confirm timezone, precision, nullability, and output format expectations.

# Under the Hood

Fields are bound to a parent serializer and source. Input skips read-only fields and uses defaults/required checks; output skips write-only fields. Field-level validators run before object-level validation.

# Performance Notes

Large list/dict fields and deeply nested fields increase validation and serialization cost. Bound collection sizes and query related records efficiently.

# Best Practices

- Match field type to API semantics, not just convenient Python type.
- Set length/range/choice limits for user-controlled input.
- Keep read/write direction explicit.
- Validate identifiers and permissions separately; a valid ID is not authorization.

# Common Mistakes

- Confusing optional (`required=False`) with nullable (`allow_null=True`).
- Confusing blank strings with null values.
- Treating format validation as proof of real-world truth.
- Assuming a relation key grants access.

# Debugging

Inspect declared/generated field properties with `repr(serializer)`. Check whether failures arise from coercion, requiredness, validators, relation lookup, or object-level rules.

# Interview Questions

1. What does a serializer field do? Converts and validates values for input/output.
2. `required=False` vs `allow_null=True`? Omission allowed vs explicit null allowed.
3. What does `read_only` mean? Included in output but ignored as client-writable input.
4. Does EmailField verify ownership? No.

# Practice Exercises

1. Define fields for product ID, quantity, decimal price, and optional note.
2. Test missing, null, blank, too-long, and out-of-range inputs.
3. Add a read-only timestamp field and verify client input cannot alter it.

# Chapter Summary

Fields are the serializer's type and validation units. Choose precise classes and options, test boundary cases, and keep data-format validation distinct from authorization and business truth.

# Introduction

Field-level validation checks one field independently. DRF supports field options, validator functions, and a `validate_<field_name>(value)` method on the serializer.

# Why this concept exists

Rules such as maximum length, allowed range, or code format belong close to the field they constrain. Centralizing them prevents different endpoints from accepting inconsistent values.

# Real-world analogy

At a checkpoint, each ticket is checked for its own validity before anyone checks whether the whole group is arranged correctly.

# Internal Working

```text
field primitive -> conversion -> field validators -> validate_field(value)
 -> validated value enters attrs
```

The method name must match the serializer field. It receives the converted Python value and must return it or raise `serializers.ValidationError`.

# Request Lifecycle (where applicable)

During `serializer.is_valid()`, DRF processes each writable field and invokes field-level validation before object-level `validate(attrs)`.

# Syntax

```python
def validate_code(self, value):
    if not value.startswith("PRD-"):
        raise serializers.ValidationError("Code must start with PRD-.")
    return value
```

# Complete Example 1

This endpoint validates a product code and returns normalized input without persistence.

```python
# api/serializers.py
from rest_framework import serializers


class ProductCodeSerializer(serializers.Serializer):
    code = serializers.CharField(max_length=20, trim_whitespace=True)

    def validate_code(self, value):
        normalized = value.upper()
        if not normalized.startswith("PRD-"):
            raise serializers.ValidationError("Code must start with PRD-.")
        suffix = normalized.removeprefix("PRD-")
        if not suffix.isalnum():
            raise serializers.ValidationError(
                "The code suffix must contain only letters and numbers."
            )
        return normalized
```

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .serializers import ProductCodeSerializer


@api_view(["POST"])
def validate_product_code(request):
    serializer = ProductCodeSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    return Response({"accepted_code": serializer.validated_data["code"]})
```

```python
# api/urls.py
from django.urls import path
from .views import validate_product_code

urlpatterns = [path("codes/validate/", validate_product_code)]
```

**Line-by-line:** `CharField` trims surrounding whitespace and limits length. `validate_code` receives the converted string, normalizes case, checks the required prefix and suffix characters, then returns the normalized value. The view binds request data, validates, and returns the cleaned code.

**Request → Serializer → Validation → Database → Response:** request JSON -> parser -> CharField conversion -> `validate_code` -> no database in this validation endpoint -> response. **Internal working:** A raised field validation error is associated with `code`; returning a value replaces that field in `validated_data`. **Professional use case:** Normalize human-entered identifiers consistently. Removing the return means the field cannot contribute its validated value; removing the prefix check accepts codes outside the contract.

# Complete Example 2

A standalone validator can be reused:

```python
from rest_framework import serializers


def must_be_even(value):
    if value % 2:
        raise serializers.ValidationError("Value must be even.")


class EvenValueSerializer(serializers.Serializer):
    value = serializers.IntegerField(validators=[must_be_even])
```

The validator receives the converted integer. Reuse the function on multiple fields when the rule is identical. It should not mutate external state.

# Complete Example 3

Test field validation directly:

```python
serializer = ProductCodeSerializer(data={"code": "prd-a17"})
assert serializer.is_valid()
assert serializer.validated_data["code"] == "PRD-A17"
```

This asserts normalization and accepted shape. Also test rejected prefix, invalid suffix, and maximum length through the field error.

# Advanced Example

For uniqueness, DRF may generate a validator from model metadata, but concurrent requests can both pass before either inserts. Keep a database unique constraint and handle a conflict safely at the persistence boundary.

# Under the Hood

Field conversion runs before `validate_<field>`, so an integer validator receives an integer rather than the original JSON string. Field-level exceptions are collected into the serializer's error mapping.

# Performance Notes

Pure string/range validation is cheap. Database or network checks are not; avoid one query per field in bulk serialization.

# Best Practices

- Use built-in field options before writing custom validation.
- Name methods exactly `validate_<field>`.
- Return the normalized value.
- Keep validators deterministic and side-effect-free.

# Common Mistakes

- Typo in method name means validation never runs.
- Validating raw types incorrectly after field conversion.
- Forgetting to return a normalized value.
- Treating uniqueness validation as concurrency protection.

# Debugging

Print/inspect serializer representation and test `is_valid()`, `errors`, and `validated_data`. Confirm the method name corresponds to a declared writable field.

# Interview Questions

1. What argument does field validation receive? The converted field value.
2. How report a field error? Raise `serializers.ValidationError`.
3. Why return from `validate_<field>`? The returned value becomes the validated field value.
4. Where place cross-field rules? `validate(attrs)`.

# Practice Exercises

1. Normalize a SKU to uppercase and enforce a prefix.
2. Add an even-number reusable validator.
3. Test accepted, normalized, and rejected values.

# Chapter Summary

Field validation protects and normalizes one value. Use built-in options where possible; otherwise implement `validate_<field>`, return the cleaned value, and raise DRF validation errors.

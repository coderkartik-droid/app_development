# Introduction

A custom validator is a callable that checks a value and raises `serializers.ValidationError` when it violates a rule. Validators can be attached to fields and reused across serializers.

# Why this concept exists

Repeated domain rules—such as a reserved word, identifier format, or numeric range—should have one tested implementation rather than slightly different copies in multiple serializers.

# Real-world analogy

A reusable validator is a standard inspection gauge used by multiple production lines to check the same dimension.

# Internal Working

DRF calls each configured validator with the converted value. A validator returns `None` when valid or raises `ValidationError` with a message/code when invalid.

# Request Lifecycle (where applicable)

```text
request.data -> field conversion -> custom validator(value)
             -> continue OR ValidationError -> DRF error response
```

# Syntax

```python
def validate_code(value):
    if not value.startswith("INV-"):
        raise serializers.ValidationError("Code must start with INV.")
```

# Complete Example 1

Validate an inventory code with a reusable function and use it in an API:

```python
# api/validators.py
from rest_framework import serializers


def validate_inventory_code(value):
    normalized = value.upper()
    if not normalized.startswith("INV-"):
        raise serializers.ValidationError(
            "Code must start with INV-.",
            code="invalid_prefix",
        )
    suffix = normalized.removeprefix("INV-")
    if len(suffix) < 4 or not suffix.isalnum():
        raise serializers.ValidationError(
            "Code suffix must contain at least four letters or digits.",
            code="invalid_suffix",
        )
```

```python
# api/serializers.py
from rest_framework import serializers
from .validators import validate_inventory_code


class InventoryInputSerializer(serializers.Serializer):
    code = serializers.CharField(max_length=20, validators=[validate_inventory_code])
```

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .serializers import InventoryInputSerializer


@api_view(["POST"])
def check_inventory_code(request):
    serializer = InventoryInputSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    return Response(serializer.validated_data)
```

**Line-by-line:** The validator normalizes for checking, validates prefix and suffix, and raises field errors with stable codes. The serializer attaches it to a bounded `CharField`. The view parses/validates and returns the submitted validated mapping. **Request → Serializer → Validation → Database → Response:** parse -> convert field -> validator executes -> no database here -> response. **Internal working:** The callable must accept the field value; it should not require hidden request state. **Professional use case:** Shared identifier policy across create and import serializers. Removing the validator from the field means it never runs; to return normalized data, use `validate_code` or a custom field because a validator's return value is not used as replacement data.

# Complete Example 2

A parameterized validator class:

```python
from rest_framework import serializers


class MaximumWords:
    def __init__(self, limit):
        self.limit = limit

    def __call__(self, value):
        if len(value.split()) > self.limit:
            raise serializers.ValidationError(
                f"Use no more than {self.limit} words."
            )


class SummarySerializer(serializers.Serializer):
    summary = serializers.CharField(validators=[MaximumWords(40)])
```

The class stores configuration and is callable by DRF. This makes the rule reusable at several limits. Keep validator instances stateless after construction.

# Complete Example 3

Use Django's built-in validators where suitable:

```python
from django.core.validators import MinValueValidator
from rest_framework import serializers


class RatingSerializer(serializers.Serializer):
    score = serializers.IntegerField(validators=[MinValueValidator(1)])
```

The integer field converts input; validator then enforces the minimum. The built-in validator is more consistent than custom duplication for standard constraints.

# Advanced Example

Unique validation often involves a database query and still cannot guarantee concurrency safety. Pair it with a database unique constraint; catch expected `IntegrityError` at a transaction boundary and map it to a safe API conflict/error.

# Under the Hood

DRF field `run_validation` calls conversion then validators. Exceptions are normalized into serializer errors. Validator callables should not mutate input or perform external side effects.

# Performance Notes

Pure validators are cheap. Query-backed validators can add one database round trip per object; avoid them in bulk/list serialization and test query counts.

# Best Practices

- Reuse validators for stable rules.
- Use built-ins for common validation.
- Raise DRF/Django validation errors, not generic exceptions.
- Back concurrency-sensitive rules with database constraints.

# Common Mistakes

- Expecting a validator's return value to transform the field.
- Forgetting to attach the validator.
- Performing writes or external calls during validation.
- Relying on uniqueness validators alone.

# Debugging

Test validator callables directly, then test through the serializer. Inspect field errors and verify callable signature/value type.

# Interview Questions

1. What does a validator return on success? Usually nothing (`None`).
2. How does it report failure? Raises a validation error.
3. Can it normalize a value? Not by returning a replacement; use `validate_<field>` or custom field.
4. How ensure uniqueness under concurrency? Database constraint plus conflict handling.

# Practice Exercises

1. Write a reusable code-prefix validator and test error codes.
2. Build a configurable maximum-words validator.
3. Compare custom logic to built-in `MinValueValidator`.

# Chapter Summary

Validators are reusable checks called during field validation. Keep them pure, attach them explicitly, and distinguish rejection from normalization; database constraints remain essential for race-safe invariants.

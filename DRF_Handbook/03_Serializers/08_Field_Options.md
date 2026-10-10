# Introduction

Field options configure whether a value is required, nullable, blank, defaulted, bounded, or read-only. These options define the details of a serializer's input contract.

# Why this concept exists

A field's type alone does not describe whether it may be omitted, set to null, contain an empty string, or exceed a size. Explicit options make those cases predictable.

# Real-world analogy

A form field's label is only part of its rules; it also needs a required marker, allowed length, and instructions for empty values.

# Internal Working

DRF field validation applies defaults and requiredness, converts input types, checks null/blank constraints, and then runs validators. ModelSerializer may infer options from model metadata, but explicit declarations override inferred behavior.

# Request Lifecycle (where applicable)

```text
request.data -> field options/conversion -> field validators -> serializer.errors or validated_data
```

# Syntax

```python
title = serializers.CharField(
    required=True,
    allow_blank=False,
    max_length=120,
)
```

# Complete Example 1

A serializer demonstrates several common field options:

```python
# api/serializers.py
from rest_framework import serializers


class ProfileInputSerializer(serializers.Serializer):
    display_name = serializers.CharField(
        required=True,
        allow_blank=False,
        max_length=80,
        trim_whitespace=True,
    )
    bio = serializers.CharField(
        required=False,
        allow_blank=True,
        allow_null=True,
        max_length=500,
        default="",
    )
    active = serializers.BooleanField(default=True)
```

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .serializers import ProfileInputSerializer


@api_view(["POST"])
def validate_profile(request):
    serializer = ProfileInputSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    return Response(serializer.validated_data)
```

**Line-by-line:** `display_name` is required, nonblank, trimmed, and bounded. `bio` may be omitted, blank, or explicitly null; if omitted the default applies. `active` defaults to true. The view binds parsed data, validates, and returns typed values. **Flow:** request -> parser -> option handling/validation -> no database in this demonstration -> response. **Internal working:** `required=False`, `allow_null=True`, and `allow_blank=True` govern different input states. **Professional tip:** Avoid contradictory configurations; choose one clear policy for optional text. Removing `max_length` leaves the API open to unbounded input.

# Complete Example 2

A decimal field:

```python
amount = serializers.DecimalField(
    max_digits=9,
    decimal_places=2,
    min_value=0,
)
```

It validates precision and a lower bound while representing decimal values according to DRF settings. For money, also define currency and rounding semantics; field configuration alone does not define the full financial model.

# Complete Example 3

Partial update changes requiredness for omitted fields:

```python
serializer = ProfileSerializer(instance=profile, data=request.data, partial=True)
serializer.is_valid(raise_exception=True)
serializer.save()
```

`partial=True` makes fields not provided by the client optional for this validation pass. Explicitly submitted `null` still depends on `allow_null`. Without `partial=True`, required fields may produce errors on PATCH.

# Advanced Example

`default`, `initial`, `source`, `validators`, and `error_messages` shape field behavior. A default is not equivalent to a model default in every serializer mode; test omitted, null, and partial-update cases explicitly.

# Under the Hood

During `run_validation`, fields handle empty values, apply defaults where appropriate, convert via `to_internal_value`, and invoke validators. Errors are normalized into DRF's structured error objects.

# Performance Notes

Simple options are inexpensive; expensive validators (database uniqueness queries, external lookups) can dominate. Keep external I/O out of field validators.

# Best Practices

- Specify lengths/ranges and null/blank policy.
- Keep defaults deterministic and safe.
- Test create and partial update separately.
- Enforce critical invariants with database constraints too.

# Common Mistakes

- Treating missing, null, and blank as identical.
- Assuming PATCH automatically allows null.
- Adding a default that masks required client input.
- Accepting unbounded strings or numeric values.

# Debugging

Inspect generated field options and error keys. Test each input state individually: omitted, `null`, empty string, whitespace, wrong type, and boundary values.

# Interview Questions

1. What does `required=False` allow? Field omission.
2. What does `allow_null=True` allow? Explicit null value.
3. What does `allow_blank=True` allow? Empty string for text.
4. What does `partial=True` do? Allows omitted fields for that serializer operation.

# Practice Exercises

1. Define optional nullable description with a maximum size.
2. Test missing, null, blank, and whitespace-only values.
3. Compare full update validation with PATCH.

# Chapter Summary

Field options define edge behavior beyond the field type. Distinguish omission, null, and blank, bound user input, and test defaults and partial updates explicitly.

# Introduction

Custom serializer fields implement conversion or representation rules not covered by built-in DRF fields. Before writing one, check whether a built-in field, `source`, or a small validation method already solves the problem clearly.

# Why this concept exists

Some domain values use a special primitive representation or require consistent conversion across many serializers. A custom field centralizes that mapping.

# Real-world analogy

A custom field is a specialist translator for a domain-specific notation that ordinary forms do not understand.

# Internal Working

Input conversion is implemented in `to_internal_value(data)`, which should return a native value or raise `serializers.ValidationError`. Output conversion is implemented in `to_representation(value)`.

# Request Lifecycle (where applicable)

```text
request primitive -> to_internal_value -> validated value -> persistence
instance value -> to_representation -> primitive -> renderer
```

# Syntax

```python
class UppercaseCodeField(serializers.CharField):
    def to_internal_value(self, data):
        value = super().to_internal_value(data)
        return value.upper()
```

# Complete Example 1

This custom field accepts a list of integer coordinates and represents them as a comma-separated string. It is illustrative; a real geographic system should use a well-defined coordinate standard.

```python
# api/serializers.py
from rest_framework import serializers


class CoordinatePairField(serializers.Field):
    def to_internal_value(self, data):
        if not isinstance(data, list) or len(data) != 2:
            raise serializers.ValidationError(
                "Expected a two-item list: [x, y]."
            )
        try:
            x, y = (float(value) for value in data)
        except (TypeError, ValueError):
            raise serializers.ValidationError(
                "Both coordinate values must be numeric."
            )
        if not (-180 <= x <= 180 and -90 <= y <= 90):
            raise serializers.ValidationError("Coordinates are out of range.")
        return (x, y)

    def to_representation(self, value):
        x, y = value
        return f"{x},{y}"


class LocationSerializer(serializers.Serializer):
    coordinates = CoordinatePairField()
```

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .serializers import LocationSerializer


@api_view(["POST"])
def normalize_location(request):
    serializer = LocationSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    point = serializer.validated_data["coordinates"]
    output = LocationSerializer({"coordinates": point})
    return Response(output.data)
```

```python
# api/urls.py
from django.urls import path
from .views import normalize_location

urlpatterns = [path("locations/normalize/", normalize_location)]
```

**Line-by-line:** `CoordinatePairField` subclasses `Field`. `to_internal_value` validates list shape, converts each item to float, and checks geographic ranges; invalid input raises a field validation error. It returns a Python tuple. `to_representation` converts that tuple to a string. `LocationSerializer` declares the custom field. The view validates input, extracts typed data, reuses serializer in output mode, and returns a response.

**Request → Serializer → Validation → Database → Response:** request list -> custom conversion/range validation -> this example has no database -> output conversion -> JSON rendering. **Internal working:** Input and output transformations are separate methods. **Professional tip:** Prefer a JSON array representation unless a client contract explicitly requires a string; output formats should be intuitive. Removing range checks permits invalid coordinates.

# Complete Example 2

A more typical custom character field can normalize a code while preserving standard CharField validation:

```python
class NormalizedCodeField(serializers.CharField):
    def to_internal_value(self, data):
        value = super().to_internal_value(data)
        return value.strip().upper()
```

Subclassing `CharField` retains its type/length behavior, then normalizes. Do not normalize values when case or whitespace is semantically meaningful.

# Complete Example 3

Use a custom field test without an HTTP view:

```python
field = NormalizedCodeField(max_length=8)
assert field.run_validation(" ab12 ") == "AB12"
```

This tests conversion directly. Also test invalid types and lengths; serializer-level tests ensure field errors are attached to the expected key.

# Advanced Example

For database relations, prefer DRF's related fields and queryset restrictions instead of custom lookup code. If a field performs database access, ensure lookup scope enforces authorization and avoid one query per item in list serialization.

# Under the Hood

The parent serializer binds the field, passes incoming primitive values to `run_validation`, and collects converted values. On output, field representation obtains the source attribute and calls `to_representation`.

# Performance Notes

Custom conversions run per object and field. Avoid expensive work, network calls, or unbounded regular expressions inside conversion methods.

# Best Practices

- Subclass the closest built-in field when possible.
- Raise `serializers.ValidationError` for invalid values.
- Make input and output representation explicit and symmetric where appropriate.
- Test edge cases and document the wire format.

# Common Mistakes

- Implementing only one direction when both are needed.
- Raising generic exceptions instead of validation errors.
- Performing side effects in a field.
- Duplicating built-in functionality unnecessarily.

# Debugging

Call `run_validation` in focused tests; inspect serializer errors and output. Confirm source values match the type expected by `to_representation`.

# Interview Questions

1. Which method converts input? `to_internal_value`.
2. Which method converts output? `to_representation`.
3. What should invalid input raise? `serializers.ValidationError`.
4. Why prefer built-in fields? They provide conventional conversion and well-tested validation.

# Practice Exercises

1. Create a field normalizing a fixed-format product code.
2. Test type, boundary, and representation behavior.
3. Compare a custom field with built-in `RegexField`/validators.

# Chapter Summary

Custom fields are for reusable domain-specific conversion, not a first resort. Separate input and output transformations, raise DRF validation errors, and keep fields deterministic and efficient.

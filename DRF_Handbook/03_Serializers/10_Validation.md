# Introduction

Serializer validation checks untrusted input before application code persists or acts on it. DRF combines field conversion, built-in validators, field-level methods, and object-level validation.

# Why this concept exists

Clients can send missing, malformed, oversized, or contradictory values. Central validation makes errors consistent and prevents invalid data from reaching business logic.

# Real-world analogy

Validation is quality control at a factory entrance: inspect each component and then verify the assembled product makes sense before production continues.

# Internal Working

```text
is_valid()
  -> each writable field: empty/default handling -> conversion -> validators
  -> serializer.validate_<field>()
  -> serializer.validate(attrs)
  -> validated_data OR errors
```

`is_valid(raise_exception=True)` raises DRF's `ValidationError`; DRF's exception handler normally turns it into a 400 response.

# Request Lifecycle (where applicable)

```text
request bytes -> parser -> request.data -> serializer(data=...)
 -> field validation -> cross-field validation -> save/response
```

# Syntax

```python
serializer = InputSerializer(data=request.data)
serializer.is_valid(raise_exception=True)
clean_values = serializer.validated_data
```

# Complete Example 1

This input serializer validates a date range and a bounded label; the endpoint returns validated data without storing it.

```python
# api/serializers.py
from rest_framework import serializers


class ReportRequestSerializer(serializers.Serializer):
    label = serializers.CharField(max_length=60)
    start_year = serializers.IntegerField(min_value=2000, max_value=2100)
    end_year = serializers.IntegerField(min_value=2000, max_value=2100)

    def validate(self, attrs):
        if attrs["end_year"] < attrs["start_year"]:
            raise serializers.ValidationError({
                "end_year": "Must be the same as or later than start_year."
            })
        return attrs
```

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .serializers import ReportRequestSerializer


@api_view(["POST"])
def report_preview(request):
    serializer = ReportRequestSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    return Response({"filters": serializer.validated_data})
```

```python
# api/urls.py
from django.urls import path
from .views import report_preview

urlpatterns = [path("reports/preview/", report_preview, name="report-preview")]
```

**Line-by-line:** Fields convert strings to integers and enforce bounds. `validate(attrs)` performs a cross-field comparison and returns the mapping unchanged if valid. The view binds parsed body data, invokes validation, then returns cleaned values. **Request → Serializer → Validation → Database → Response:** parser creates primitives -> serializer validates field rules -> object rule checks year order -> no database in preview -> Response renders JSON. **Internal working:** Field errors and object errors are accumulated into structured errors. **Professional tip:** Validate shape here, but enforce business rules again in the authoritative domain/database layer when concurrent writes can violate them. Removing `return attrs` breaks object validation's contract.

# Complete Example 2

Use exception-based validation:

```python
serializer = ReportRequestSerializer(data=request.data)
serializer.is_valid(raise_exception=True)
```

DRF handles the raised validation exception and returns its structured error response. This avoids manually branching on `.errors` in every view. Do not use this form if you intentionally want to combine errors into a custom response.

# Complete Example 3

Validate manually and inspect errors:

```python
serializer = ReportRequestSerializer(data=request.data)
if not serializer.is_valid():
    return Response({"errors": serializer.errors}, status=400)
```

This makes the response envelope explicit. `.validated_data` should only be used on the valid branch.

# Advanced Example

For transactional domain checks—such as uniqueness under concurrent requests—serializer validation alone may race. Back it with database constraints and convert known integrity violations into an appropriate conflict/validation response without leaking SQL details.

# Under the Hood

DRF calls `run_validation` on writable fields, gathers field errors, then calls serializer-level `validate`. `is_valid` stores either `_validated_data` or `_errors`, and only one mode is available after validation.

# Performance Notes

Database-backed uniqueness validators add queries. Avoid remote calls in validation; keep lookup counts bounded and profile list serializers separately.

# Best Practices

- Keep validators pure and deterministic.
- Validate types, limits, and cross-field invariants.
- Use database constraints for concurrency-sensitive invariants.
- Return actionable, stable error structures.

# Common Mistakes

- Reading validated data without checking validation.
- Saving or sending messages from validators.
- Swallowing validation errors.
- Assuming a uniqueness check removes race conditions.

# Debugging

Inspect `serializer.errors` and identify field vs non-field keys. Reproduce each constraint independently; check whether parsing fails before serializer validation.

# Interview Questions

1. What happens in `is_valid()`? Field conversion/validators and object validation run.
2. Where is cleaned input? `validated_data`.
3. What does `raise_exception=True` do? Raises DRF validation exception for standard handling.
4. Why also use DB constraints? They remain authoritative under concurrent writes and alternate write paths.

# Practice Exercises

1. Add a cross-field year/date check.
2. Test missing, malformed, boundary, and contradictory values.
3. Add an API test asserting validation errors return 400.

# Chapter Summary

Serializer validation converts and checks input before side effects. Combine field and object rules, keep validation pure, and use database constraints for invariants that must survive concurrency.

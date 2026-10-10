# Introduction

Serializer mistakes usually involve using the wrong mode, misunderstanding validation order, exposing fields unintentionally, or assuming validation automatically persists or authorizes data.

# Why this concept exists

Serializers sit between untrusted clients and application data. A small misunderstanding can lead to runtime errors, invalid records, or data exposure.

# Real-world analogy

A form that accepts the wrong field, skips review, or prints confidential values can fail even if it looks professionally designed.

# Internal Working

```text
data vs instance mode -> validation -> create/update -> representation
             permissions and database constraints remain separate layers
```

# Request Lifecycle (where applicable)

DRF parses the request, serializer validates, view/service performs authorized persistence, and serializer output becomes the response. Bugs often come from skipping or conflating one of these steps.

# Syntax

```python
serializer = ItemSerializer(data=request.data)
serializer.is_valid(raise_exception=True)
item = serializer.save()
```

# Complete Example 1

Correctly validate before accessing cleaned data:

```python
# api/serializers.py
from rest_framework import serializers


class TagInputSerializer(serializers.Serializer):
    name = serializers.CharField(max_length=50)
```

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .serializers import TagInputSerializer


@api_view(["POST"])
def validate_tag(request):
    serializer = TagInputSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    return Response({"name": serializer.validated_data["name"]})
```

```python
# api/urls.py
from django.urls import path
from .views import validate_tag

urlpatterns = [path("tags/validate/", validate_tag)]
```

**Line-by-line:** The serializer declares a bounded string. The view initializes in input mode with `data=`, validates with exception handling, then reads `validated_data`. **Request → Serializer → Validation → Database → Response:** parse -> field checks -> no database for validation preview -> Response. **Internal working:** `validated_data` is guarded until `is_valid()` runs successfully. **Professional use case:** Validate command payloads consistently. Removing the validation call makes `validated_data` access fail; using `serializer.data` as a substitute can expose an unvalidated representation.

# Complete Example 2

Use `instance` to serialize output:

```python
serializer = TagSerializer(instance=tag)
return Response(serializer.data)
```

This is representation mode, not input validation. Do not pass the model instance as `data`; that causes DRF to treat it as primitive input.

# Complete Example 3

Protect fields:

```python
class Meta:
    model = UserProfile
    fields = ["id", "display_name", "is_staff"]
    read_only_fields = ["id", "is_staff"]
```

This blocks client writes through this serializer, but permissions and server-side policy still govern the operation. It also intentionally exposes `is_staff` in output, which may be undesirable; remove it if clients should not see it.

# Advanced Example

A validator that queries for uniqueness is not race-proof. Keep a database unique constraint, handle integrity failures intentionally, and avoid catching `Exception` as a generic validation fallback.

# Under the Hood

Input serializers have `initial_data`; output serializers have an `instance`; after validation, `validated_data` or `errors` is set. `.data` represents output and may be cached, so don't mutate instance/data expecting recalculation.

# Performance Notes

Avoid query-heavy validators and method fields. Bulk serialization multiplies per-object costs; optimize querysets before serializing.

# Best Practices

- Use correct initialization mode.
- Validate before save or reading validated values.
- Explicitly allowlist fields and directions.
- Scope database access by permissions/ownership.

# Common Mistakes

- Calling `.save()` before `.is_valid()`.
- Using `.data` as validated input.
- Forgetting to return `attrs` from `validate`.
- Assuming read-only equals authorized.
- Assuming nested writes are automatic.

# Debugging

Print serializer representation; check whether it has `instance`, `initial_data`, `validated_data`, or `errors`. Trace output exposure and database writes independently.

# Interview Questions

1. How identify input mode? Serializer constructed with `data=`.
2. When use `.data`? To obtain serialized representation.
3. When use `.validated_data`? After successful validation.
4. Does read-only protect endpoint access? No.

# Practice Exercises

1. Fix an example that accesses validated data before validation.
2. Remove an unnecessarily exposed model field.
3. Test invalid input, permission failure, and database conflict separately.

# Chapter Summary

Use the correct serializer mode and validation sequence. Keep representation, validation, persistence, and authorization distinct; most serious serializer bugs arise from collapsing these responsibilities.

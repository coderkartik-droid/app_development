# Introduction

Serializers bridge API representations and Python objects. They define which values clients may send, which values they may see, how values are converted, and which validation rules apply.

# Why this concept exists

Directly returning models leaks implementation details; manually checking every input duplicates validation and produces inconsistent errors. Serializers create a reusable, testable boundary around both problems.

# Real-world analogy

A serializer is a customs declaration: it checks the submitted item against rules, transforms it into the internal inventory form, and controls what details are shown on the outbound label.

# Internal Working

```text
Input: parsed primitives -> fields -> validators -> validated_data -> create/update
Output: model/object -> fields -> primitive representation -> renderer
```

Serializer fields perform conversion and validation; object validation handles cross-field rules. Model persistence occurs only through `save()` or explicit application code.

# Request Lifecycle (where applicable)

The parser converts body bytes; the serializer validates and transforms values; the view applies authorization and persistence; the output serializer and renderer create the response.

# Syntax

```python
serializer = ItemSerializer(data=request.data)
serializer.is_valid(raise_exception=True)
item = serializer.save()
```

# Complete Example 1

The example provides an allowlisted model representation and a validated create API. Assume `Item` has `name` and `description`, and `api` is installed.

```python
# api/serializers.py
from rest_framework import serializers
from .models import Item


class ItemSerializer(serializers.ModelSerializer):
    class Meta:
        model = Item
        fields = ["id", "name", "description"]
        read_only_fields = ["id"]
```

```python
# api/views.py
from rest_framework import generics
from rest_framework.permissions import IsAuthenticated
from .models import Item
from .serializers import ItemSerializer


class ItemCreateView(generics.CreateAPIView):
    queryset = Item.objects.all()
    serializer_class = ItemSerializer
    permission_classes = [IsAuthenticated]
```

```python
# api/urls.py
from django.urls import path
from .views import ItemCreateView

urlpatterns = [path("items/", ItemCreateView.as_view(), name="item-create")]
```

**Line-by-line:** `ModelSerializer` derives common fields and persistence behavior from `Item`. `fields` is the public allowlist. `read_only_fields` prevents client assignment of `id`. `CreateAPIView` supplies POST behavior; queryset establishes the resource source; serializer class links validation/output; permission class requires authentication. URL config exposes the endpoint.

**Request → Serializer → Validation → Database → Response:** request body is parsed -> serializer validates types and model-derived constraints -> create flow saves through ORM -> response serializes the new object with 201. **Internal working:** The generic view calls serializer validation and `perform_create`/save sequence. Removing `permission_classes` may expose the endpoint according to global defaults; never infer security from the form.

**Professional use case:** Authenticated users create records using a stable, restricted input/output contract.

# Complete Example 2

Use a plain `Serializer` when the input is not a direct model shape:

```python
from rest_framework import serializers


class ReportRangeSerializer(serializers.Serializer):
    start_year = serializers.IntegerField(min_value=2000)
    end_year = serializers.IntegerField(min_value=2000)

    def validate(self, attrs):
        if attrs["end_year"] < attrs["start_year"]:
            raise serializers.ValidationError("End year must not precede start year.")
        return attrs
```

This validates a query command rather than creating a model row. It demonstrates why serializers are useful even without persistence: type conversion, field checks, and cross-field rules are centralized. `save()` is not available for meaningful persistence unless `create()`/`update()` are implemented.

# Complete Example 3

Test serializer validation directly:

```python
serializer = ReportRangeSerializer(data={"start_year": "2024", "end_year": 2022})
assert serializer.is_valid() is False
assert "non_field_errors" in serializer.errors
```

DRF converts the numeric string to an integer during field validation, then object validation detects the invalid range. Direct tests are fast; also test the HTTP view for its status and response shape.

# Advanced Example

Treat serializers as contract code. A field rename or nullability change can break clients even if the database migration succeeds. Introduce compatibility consciously and test representative old client payloads.

# Under the Hood

`Serializer` field binding associates names, parent serializer, validators, and defaults. Validation converts primitive values, checks fields, then calls object-level `validate`. Representation iterates readable fields and calls their representation logic.

# Performance Notes

Avoid overly nested serializers and computed fields that issue queries. Query related objects in the view/queryset and profile serializer output for large lists.

# Best Practices

- Use plain serializers for commands/non-model payloads and model serializers for common model mapping.
- Expose only approved fields.
- Keep validation deterministic and side-effect-free.
- Keep saving and business transactions explicit.

# Common Mistakes

- Thinking every serializer must map to a model.
- Saving in a validator.
- Trusting client-supplied owner or permission fields.
- Testing only serializer internals and not HTTP behavior.

# Debugging

Inspect `errors` first; verify field types, required/null/blank options, and whether `validate()` returns the attrs mapping. Confirm view actually uses the intended serializer class.

# Interview Questions

1. Why serializers? Conversion, validation, and explicit API contracts.
2. ModelSerializer vs Serializer? Model metadata/default persistence versus fully explicit fields/behavior.
3. Does validation write to database? No.
4. What handles cross-field checks? Object-level `validate(attrs)`.

# Practice Exercises

1. Define a serializer for a date range and reject reversed bounds.
2. Add a read-only output field to a model serializer.
3. Test malformed input, valid input, and HTTP status separately.

# Chapter Summary

Serializers prevent ad hoc input/output code and make API contracts explicit. They validate and convert; persistence remains a deliberate step, with permissions and database constraints protecting invariants.

# Introduction

Object-level validation checks relationships among multiple input fields. In a serializer, implement `validate(self, attrs)` to enforce rules that cannot be decided by validating each field in isolation.

# Why this concept exists

Each field can be individually valid while the submitted object is contradictory—for example, an end date before a start date or a minimum price higher than a maximum.

# Real-world analogy

Every ingredient may pass its own inspection, but the full recipe still needs a final check that quantities and steps agree.

# Internal Working

DRF first converts and validates individual writable fields. It then passes the resulting mapping to `validate(attrs)`. Return the mapping or raise `serializers.ValidationError`.

# Request Lifecycle (where applicable)

```text
request.data -> fields validate independently -> validate(attrs)
             -> validated_data OR object/field errors -> view save/response
```

# Syntax

```python
def validate(self, attrs):
    if attrs["end"] <= attrs["start"]:
        raise serializers.ValidationError({"end": "Must follow start."})
    return attrs
```

# Complete Example 1

Validate a booking time range. This endpoint demonstrates validation without database persistence.

```python
# api/serializers.py
from rest_framework import serializers


class BookingRequestSerializer(serializers.Serializer):
    start_minute = serializers.IntegerField(min_value=0, max_value=1439)
    end_minute = serializers.IntegerField(min_value=0, max_value=1439)
    party_size = serializers.IntegerField(min_value=1, max_value=12)

    def validate(self, attrs):
        if attrs["end_minute"] <= attrs["start_minute"]:
            raise serializers.ValidationError({
                "end_minute": "Must be later than start_minute."
            })
        return attrs
```

```python
# api/views.py
from rest_framework.decorators import api_view
from rest_framework.response import Response
from .serializers import BookingRequestSerializer


@api_view(["POST"])
def booking_preview(request):
    serializer = BookingRequestSerializer(data=request.data)
    serializer.is_valid(raise_exception=True)
    return Response({"booking": serializer.validated_data})
```

```python
# api/urls.py
from django.urls import path
from .views import booking_preview

urlpatterns = [path("bookings/preview/", booking_preview)]
```

**Line-by-line:** Integer fields convert and bound values. `validate` compares the start and end after field checks; an error is attached to `end_minute`; valid attrs are returned. The view validates before returning a preview. **Request → Serializer → Validation → Database → Response:** parser -> field checks -> cross-field check -> no database -> response. **Internal working:** `attrs` contains converted Python values, not necessarily the original strings. **Professional use case:** Validate booking/filter command semantics before checking availability in a transaction. Removing `return attrs` breaks validation; placing the comparison in one field method can be awkward because the other field may not yet be present.

# Complete Example 2

Object validation can enforce a conditional requirement:

```python
class ContactChoiceSerializer(serializers.Serializer):
    method = serializers.ChoiceField(choices=["email", "phone"])
    email = serializers.EmailField(required=False)
    phone = serializers.CharField(required=False)

    def validate(self, attrs):
        if attrs["method"] == "email" and not attrs.get("email"):
            raise serializers.ValidationError({"email": "Required for email contact."})
        if attrs["method"] == "phone" and not attrs.get("phone"):
            raise serializers.ValidationError({"phone": "Required for phone contact."})
        return attrs
```

This code should be used as a complete serializer definition in the app. Field validation ensures valid choice/email format; object validation links fields. Consider partial-update semantics if reusing it for PATCH.

# Complete Example 3

Test the relationship:

```python
serializer = BookingRequestSerializer(
    data={"start_minute": 900, "end_minute": 850, "party_size": 2}
)
assert not serializer.is_valid()
assert "end_minute" in serializer.errors
```

The test verifies the cross-field rule and error location. Also test valid adjacent boundaries and field-level failures.

# Advanced Example

During partial updates, `attrs` may contain only submitted fields. For update-specific rules, merge missing values from `self.instance` carefully, or design a dedicated command serializer. Do not index an optional key assuming full input.

# Under the Hood

`validate(attrs)` runs after field-level processing and defaults. Object-level errors may use `non_field_errors` or be keyed to a field using a dictionary. Return a mapping for `validated_data`.

# Performance Notes

Cross-field comparisons are usually cheap. Keep database availability checks and external calls out of this hook where possible; they complicate latency and transaction guarantees.

# Best Practices

- Put multi-field invariants in `validate`.
- Use precise error keys and messages.
- Return attrs unchanged or deliberately normalized.
- Handle partial updates and instance values explicitly.

# Common Mistakes

- Forgetting `return attrs`.
- Accessing optional keys directly.
- Treating serializer object checks as a substitute for transactional constraints.
- Performing side effects during validation.

# Debugging

Inspect field errors first; object validation runs only after field validation succeeds. Print safe `attrs` in development and test full and partial input separately.

# Interview Questions

1. What does object-level validation receive? Converted mapping of fields (`attrs`).
2. What must it return? The validated/normalized mapping.
3. When does it run? After field-level validation succeeds.
4. Where do partial updates complicate it? Omitted fields may be absent from attrs.

# Practice Exercises

1. Validate start/end values and return a field-specific error.
2. Require one contact field based on a selected method.
3. Test same-object validation in full and partial update modes.

# Chapter Summary

Object-level validation enforces rules across fields after their individual conversion. Return `attrs`, report actionable errors, and account for omitted fields during partial updates.

# Introduction

This chapter revises serializer concepts commonly explored in DRF interviews: conversion, validation, ModelSerializer behavior, field direction, context, nested writes, and persistence methods.

# Why this concept exists

Interviewers look for understanding of data flow and failure behavior, not just memorized class names. Explain what occurs before and after `is_valid()` and `save()`.

# Real-world analogy

An interview answer is a code review: describe the rule, show a concrete example, and state what can fail if it is omitted.

# Internal Working

```text
parsed primitive -> field validation -> object validation -> validated_data
                  -> create/update -> instance -> representation -> renderer
```

# Request Lifecycle (where applicable)

The request parser creates primitives; serializer validates; view policy authorizes; persistence writes; serializer output becomes a response. A good answer identifies which layer owns each responsibility.

# Syntax

```python
serializer = BookSerializer(instance=book, data=request.data, partial=True)
serializer.is_valid(raise_exception=True)
updated = serializer.save()
```

# Complete Example 1

Interview prompt: “How do you validate and save an API input?”

```python
from rest_framework import serializers


class NameSerializer(serializers.Serializer):
    name = serializers.CharField(max_length=80)


def validate_payload(payload):
    serializer = NameSerializer(data=payload)
    serializer.is_valid(raise_exception=True)
    return serializer.validated_data
```

**Line-by-line:** The plain serializer defines a bounded field. The function constructs in input mode, validates, raises a DRF validation exception on failure, and returns converted values on success. In an API view, DRF maps the exception to a 400. **Flow:** request -> parser -> serializer -> field checks -> validated values -> application logic -> response. **Internal working:** No database write occurs in this function. **Professional tip:** State separately how persistence and authorization are performed.

# Complete Example 2

Interview prompt: “How prevent clients from assigning their owner?”

```python
class RecordSerializer(serializers.ModelSerializer):
    class Meta:
        model = Record
        fields = ["id", "title", "owner"]
        read_only_fields = ["id", "owner"]
```

Then assign owner from `request.user` in `perform_create` or a service. Read-only prevents serializer input from setting it; authenticated permission and queryset scoping still protect records.

# Complete Example 3

Interview prompt: “How avoid N+1 nested serializer queries?”

```python
def get_queryset(self):
    return Order.objects.prefetch_related("items").select_related("customer")
```

Use `prefetch_related` for many-valued relationships and `select_related` for single-valued FK/one-to-one relations where serializer reads them. Confirm query count with tests.

# Advanced Example

Practice concise answers:

- `Serializer` is explicit and model-independent; `ModelSerializer` infers model fields and common persistence.
- `validate_<field>` checks one converted field; `validate(attrs)` checks combinations.
- `create` runs without an instance; `update` receives one.
- Nested read representation is easy; nested writes require explicit policy.

# Under the Hood

DRF binds serializer fields, runs conversions/validators, stores validated data or errors, calls create/update on save, and represents the returned instance. Request/response rendering surrounds this lifecycle.

# Performance Notes

Mention query optimization and bounded collection size when discussing serializers. Computed fields may add per-object work.

# Best Practices

- Answer with execution sequence and a realistic example.
- Mention validation errors and status behavior.
- Distinguish serializer concerns from permissions and database constraints.
- State trade-offs; avoid claiming serializer rules alone guarantee concurrent invariants.

# Common Mistakes

- Saying `save()` validates automatically.
- Calling `.data` the same as `.validated_data`.
- Claiming read-only fields provide authorization.
- Assuming nested serializers save children automatically.

# Debugging

Use the lifecycle diagram, then inspect mode, field config, errors, query count, persistence method, and response shape.

# Interview Questions

1. What is serializer's primary role? Convert and validate representations.
2. What does `is_valid` return? Boolean, optionally raising on failure.
3. When does `create` vs `update` run? Based on whether serializer has an instance at `save`.
4. Why `fields` allowlist? Prevent accidental exposure and stabilize contract.
5. Where validate cross-field rules? `validate(attrs)`.
6. What does `many=True` do? Serializes/validates a collection of objects.
7. Why pass context? Supply trusted request/view metadata.
8. How handle nested writes? Explicit create/update rules and transaction.

# Practice Exercises

1. Explain a serializer request flow in 60 seconds.
2. Compare ModelSerializer and Serializer with one use case each.
3. Diagnose a query-heavy SerializerMethodField.

# Chapter Summary

Interview-ready serializer knowledge means explaining the full data flow, the validation/persistence split, field safety, nested-write ambiguity, and query performance—not only naming APIs.

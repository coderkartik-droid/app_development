# Introduction

This revision sheet summarizes serializer roles, field behavior, validation, persistence, nesting, context, and performance. Use it to trace data through a DRF endpoint.

# Why this concept exists

Serializers combine conversion, validation, and representation. Revising their boundaries prevents confusion between request parsing, authorization, persistence, and response rendering.

# Real-world analogy

A serializer is a controlled gateway: input is inspected and translated inward; output is selected and translated outward.

# Internal Working

```text
IN:  bytes -> parser -> primitives -> serializer fields -> validators -> validated_data
             -> create/update/service -> database
OUT: instance -> serializer fields -> primitives -> Response -> renderer -> bytes
```

# Request Lifecycle (where applicable)

1. URL routes request to DRF view.
2. Authentication/permissions run according to policy.
3. Parser creates `request.data` when accessed.
4. Serializer converts and validates fields and object rules.
5. View/service persists or queries.
6. Serializer represents approved output.
7. Renderer creates final response bytes.

# Syntax

```python
s = ItemSerializer(data=request.data)
s.is_valid(raise_exception=True)
item = s.save(owner=request.user)
return Response(ItemSerializer(item).data, status=201)
```

# Complete Example 1

Minimal input and output serializer pattern:

```python
from rest_framework import serializers


class LabelInputSerializer(serializers.Serializer):
    name = serializers.CharField(max_length=40)


class LabelOutputSerializer(serializers.Serializer):
    id = serializers.IntegerField()
    name = serializers.CharField()


def create_label_representation(raw_data, new_id):
    incoming = LabelInputSerializer(data=raw_data)
    incoming.is_valid(raise_exception=True)
    stored = {"id": new_id, **incoming.validated_data}
    return LabelOutputSerializer(instance=stored).data
```

**Line-by-line:** Input serializer specifies accepted data; output serializer specifies returned shape. Function validates raw mapping, combines trusted generated ID with validated name, and serializes the result in instance mode. **Request → Serializer → Validation → Database → Response:** raw parsed data -> validation -> this demonstration substitutes an in-memory ID, so no DB -> output representation. **Internal working:** Input and output contracts are independent. **Professional tip:** For real persistence replace the in-memory step with an ORM/service transaction. Removing validation before using values bypasses the contract.

# Complete Example 2

Model-backed quick reference:

```python
class ItemSerializer(serializers.ModelSerializer):
    class Meta:
        model = Item
        fields = ["id", "name", "created_at"]
        read_only_fields = ["id", "created_at"]
```

Explicit fields define what is exposed; read-only fields cannot be assigned through input. ModelSerializer supplies common persistence but not all domain rules.

# Complete Example 3

Validation method reference:

```python
def validate_name(self, value):
    if value.lower() == "admin":
        raise serializers.ValidationError("Reserved name.")
    return value

def validate(self, attrs):
    # Cross-field checks go here.
    return attrs
```

Field method receives one converted value; object method receives converted mapping. Both must report errors with DRF validation exceptions.

# Advanced Example

Quick choices:

| Requirement | Serializer tool |
|---|---|
| Model CRUD mapping | `ModelSerializer` |
| Command/non-model payload | `Serializer` |
| One-field rule | `validate_<field>` or validator |
| Cross-field rule | `validate(attrs)` |
| Computed output | `SerializerMethodField` |
| Server-only output/input | `read_only` / `write_only` |
| Related output | Nested serializer |
| Request metadata | `context` |

# Under the Hood

`is_valid()` runs conversion and validators and sets `validated_data` or `errors`. `save()` calls `create` if no instance, otherwise `update`. `.data` represents output and is not a synonym for validated input.

# Performance Notes

Paginate lists, optimize related queries, avoid expensive method fields, and inspect serializer-driven query counts.

# Best Practices

- Explicit fields, direction, bounds, and null/blank behavior.
- Validate before side effects.
- Keep authorization in permissions/querysets, not field visibility alone.
- Use transactions and database constraints for important persistence invariants.

# Common Mistakes

- `.data` vs `.validated_data` confusion.
- Saving before validation.
- Trusting client owner/price/role.
- Assuming nested writes are automatic.
- Assuming validation alone prevents races.

# Debugging

Check initialization mode, `.errors`, `.validated_data`, `.instance`, `.context`, generated field representation, queryset scope, and query count.

# Interview Questions

1. What are the serializer's two directions? Complex object to primitives; primitives to validated internal values.
2. When does create/update execute? On save, based on whether an instance exists.
3. What does read-only mean? Output but not client-writable.
4. Why use explicit field allowlists? Security and stable contracts.

# Practice Exercises

1. Explain a POST and PATCH serializer flow.
2. Implement one field validator and one cross-field validator.
3. Draw a nested transactional write and mark authorization/query boundaries.

# Chapter Summary

Serializers define and enforce representation contracts, converting input and output. They do not replace permissions, domain logic, transactions, or database constraints. Trace each operation from parser through validation and persistence to rendered response.

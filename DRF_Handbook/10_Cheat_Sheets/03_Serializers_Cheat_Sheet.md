# Serializers Cheat Sheet

## Key Concepts

- Translate model/domain values to primitives and validate incoming primitives.
- `Serializer`: explicit fields and behavior.
- `ModelSerializer`: model-derived fields and default persistence.
- `validated_data` is safe-to-use validated input, not a replacement for DB constraints.

## Important Syntax

```python
serializer = ProductSerializer(data=request.data)
serializer.is_valid(raise_exception=True)
product = serializer.save()
```

## Frequently Used Classes and Methods

| API | Use |
|---|---|
| `Serializer` | Custom/non-model data |
| `ModelSerializer` | Model CRUD |
| `CharField`, `IntegerField` | Typed fields |
| `SerializerMethodField` | Computed read-only output |
| `validate_<field>()` | Field validation |
| `validate(attrs)` | Cross-field validation |
| `create()` / `update()` | Persistence behavior |

## Common Decorators

- `@staticmethod` can define reusable validator functions.
- Validators are often passed directly: `validators=[...]`.
- Serializer validation methods are named methods, not view decorators.

## HTTP Status Codes

Serializer does not choose endpoint status: invalid input normally becomes 400; create usually 201; update usually 200; delete often 204.

## Request/Response Flow

`request.data -> serializer fields/coercion -> field validators -> object validation -> save -> serializer.data -> renderer`.

## Serializer Flow

```python
class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ["id", "name", "price"]
        read_only_fields = ["id"]
```

- Create: `Serializer(data=...)`.
- Update: `Serializer(instance, data=...)`.
- Partial: add `partial=True`.
- Multiple objects: add `many=True`.

## View/ViewSet Comparison

Serializer owns representation, field validation, and create/update conversion. View owns request orchestration, permissions, and selecting the object/queryset.

## Authentication Flow

Serializer does not authenticate. Trusted actor values are passed from view, e.g. `serializer.save(owner=request.user)`.

## JWT Flow

JWT is separate from serialization; never put access/refresh tokens into ordinary model serializers by default.

## Common Mistakes

- Calling `save()` before `is_valid()`.
- Exposing all model fields.
- Accepting owner/role/status from client.
- Using `partial=True` on create.
- Assuming serializer uniqueness checks are concurrency-safe.

## Best Practices

- Explicit `fields`.
- `read_only_fields` for server-owned values.
- Validate input and enforce critical DB constraints.
- Separate input/output serializers when contracts differ.

## Interview Revision

`is_valid()` creates `validated_data`; `save()` dispatches to `create` or `update`.

## Memory Tricks

- **I-V-S:** Instantiate, Validate, Save.
- **Field first, object second:** field validation precedes cross-field validation.

## One-Page Summary

Serializers are the API data boundary. They coerce and validate incoming values, map model instances to primitives, and optionally persist through `create()`/`update()`. Keep fields explicit and authorization outside serializer assumptions.

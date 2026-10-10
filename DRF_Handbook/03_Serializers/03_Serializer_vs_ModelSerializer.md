# Introduction

`serializers.Serializer` requires fields and persistence behavior to be defined explicitly. `serializers.ModelSerializer` can infer fields and common create/update behavior from a Django model.

# Why this concept exists

Some API payloads map closely to database records; others represent commands, calculations, or external data. One serializer style should not be forced onto every shape.

# Real-world analogy

A plain serializer is a blank form designed field by field. A model serializer is a form generated from an existing record blueprint, with room to customize its public contract.

# Internal Working

```text
Serializer: declared fields + explicit create/update (if saving)
ModelSerializer: model metadata -> inferred fields/validators + default persistence
```

Both share validation and representation APIs. ModelSerializer generation is convenient, but explicit `fields` and policy decisions remain essential.

# Request Lifecycle (where applicable)

Both receive parsed data, validate it, and can return `.errors` or `.validated_data`. A model serializer can save a model through default `create`/`update`; a plain serializer needs custom persistence methods or explicit service logic.

# Syntax

```python
class CommandSerializer(serializers.Serializer):
    count = serializers.IntegerField(min_value=1)

class RecordSerializer(serializers.ModelSerializer):
    class Meta:
        model = Record
        fields = ["id", "name"]
```

# Complete Example 1

Compare both forms for a model-backed product. Assume installed app `shop` has `Product(name, price_cents)`.

```python
# shop/serializers.py
from rest_framework import serializers
from .models import Product


class ProductModelSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ["id", "name", "price_cents"]
        read_only_fields = ["id"]


class ProductExplicitSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    name = serializers.CharField(max_length=120)
    price_cents = serializers.IntegerField(min_value=0)

    def create(self, validated_data):
        return Product.objects.create(**validated_data)

    def update(self, instance, validated_data):
        instance.name = validated_data.get("name", instance.name)
        instance.price_cents = validated_data.get(
            "price_cents", instance.price_cents
        )
        instance.save()
        return instance
```

**Line-by-line:** The model serializer declares its model and explicit public fields; DRF derives field mapping and default persistence. The plain serializer declares each field and validation constraint. Its `create` creates a model from validated values; `update` changes only supplied values and returns the saved instance.

**Request → Serializer → Validation → Database → Response:** DRF parses request data -> either serializer converts/checks fields -> `save()` calls generated or custom create/update -> ORM writes -> `.data` represents the model. **Internal working:** Both call field/object validation; only ModelSerializer infers behavior. **Professional tip:** Explicit serializer gives control for unusual rules; ModelSerializer reduces routine mapping. Removing explicit `fields` risks exposing unintended fields.

# Complete Example 2

A command serializer needs no model:

```python
from rest_framework import serializers


class RebuildIndexSerializer(serializers.Serializer):
    dry_run = serializers.BooleanField(default=True)
    max_records = serializers.IntegerField(min_value=1, max_value=5000)
```

This validates an operation request, not a persisted object. Its `validated_data` can be passed to a service. Without a model, using `ModelSerializer` would misrepresent the domain and require an artificial model.

# Complete Example 3

A read-only projection may use ModelSerializer but override a field:

```python
class ProductPublicSerializer(serializers.ModelSerializer):
    display_price = serializers.SerializerMethodField()

    class Meta:
        model = Product
        fields = ["id", "name", "display_price"]
```

This demonstrates model mapping plus custom output. The method implementation is covered in its own chapter. Ensure computed output does not perform hidden queries.

# Advanced Example

Choose by responsibility, not convenience alone: model CRUD usually fits ModelSerializer; commands and aggregates usually fit plain Serializer; complicated domain writes can validate with either and delegate persistence to a service. Keep transaction boundaries explicit.

# Under the Hood

ModelSerializer inspects model metadata to build serializer fields and default validators. Its default `create` calls the model manager; default `update` assigns validated values then saves. It does not infer every domain-specific rule or nested persistence policy.

# Performance Notes

Both serializer types have per-field conversion cost. Avoid custom methods and nested relationships that trigger per-row queries.

# Best Practices

- Explicitly list ModelSerializer fields.
- Add read-only/write-only rules for sensitive or server-owned data.
- Use plain serializers for non-model commands.
- Override create/update only when required behavior is clear and tested.

# Common Mistakes

- Assuming ModelSerializer means no validation design is needed.
- Assuming plain Serializer automatically saves.
- Overriding persistence but forgetting to return the instance.
- Exposing every model field by using `"__all__"` without review.

# Debugging

Inspect generated serializer fields with `repr(serializer)`. Check `validated_data` and identify whether `save()` dispatches to default or overridden methods.

# Interview Questions

1. What does ModelSerializer infer? Fields, some validators, and default model create/update.
2. When use plain Serializer? Non-model payloads or full control over fields/behavior.
3. Does plain Serializer save automatically? No; implement create/update or persist elsewhere.
4. Does ModelSerializer handle arbitrary nested writes? No, explicit behavior is often required.

# Practice Exercises

1. Implement one model-backed serializer in both styles.
2. Add a command serializer with cross-field validation.
3. Inspect generated ModelSerializer fields and test which are writable.

# Chapter Summary

Both serializers validate and represent data. ModelSerializer accelerates common model mapping; plain Serializer is explicit and model-independent. Select the abstraction that matches the payload and persistence responsibility.

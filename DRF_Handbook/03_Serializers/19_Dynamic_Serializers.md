# Introduction

A dynamic serializer changes its fields or behavior based on trusted runtime context, such as the request user, query option, or endpoint action. Dynamic behavior can tailor representations, but it must remain predictable and secure.

# Why this concept exists

Different callers or actions may need different field sets. A single serializer can sometimes serve these variations without duplicating most of its logic.

# Real-world analogy

A configurable report prints only the sections relevant to the reader's role, while still following one approved template.

# Internal Working

```text
view/action/context -> serializer initialization -> field policy -> validation/representation
```

Fields may be modified in `__init__`, or views may choose different serializer classes. Prefer the latter when input/output contracts differ substantially.

# Request Lifecycle (where applicable)

The view determines trusted context or action, instantiates the serializer, and then validation/representation uses its configured fields. Client-provided field selection must be allowlisted and must never bypass authorization.

# Syntax

```python
def get_serializer_class(self):
    if self.action == "list":
        return CompactSerializer
    return DetailSerializer
```

# Complete Example 1

This ModelSerializer removes an optional `description` field only when a trusted request option explicitly asks for a compact response. Assume `Product` has `name` and `description`.

```python
# catalog/serializers.py
from rest_framework import serializers
from .models import Product


class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ["id", "name", "description"]
        read_only_fields = ["id"]

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        request = self.context.get("request")
        compact = bool(request and request.query_params.get("compact") == "1")
        if compact:
            self.fields.pop("description", None)
```

```python
# catalog/views.py
from rest_framework import generics
from .models import Product
from .serializers import ProductSerializer


class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
```

```python
# catalog/urls.py
from django.urls import path
from .views import ProductListView

urlpatterns = [path("products/", ProductListView.as_view(), name="products")]
```

**Line-by-line:** Meta allowlists fields. `__init__` calls `super()` to bind the normal fields, obtains optional request context, and checks a narrow flag. It removes only the description field for compact output. Generic views provide request context. **Request → Serializer → Validation → Database → Response:** GET -> queryset loads products -> serializer fields are configured -> representation -> renderer. This is read-only; no validation/database write occurs. **Internal working:** `self.fields` is instance-local, so modifying it here does not alter the class declaration. **Professional use case:** Optional response trimming for large list cards. Removing the `super()` call breaks field initialization; never make sensitive fields appear merely because a query flag was supplied.

# Complete Example 2

Prefer a separate list serializer when shapes differ substantially:

```python
class ProductListSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ["id", "name"]


class ProductDetailSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ["id", "name", "description"]
```

Distinct serializers make public contracts obvious and independently testable. They may share field declarations through careful base classes, but avoid inheritance that obscures exposure.

# Complete Example 3

Choose serializer by action in a ViewSet:

```python
def get_serializer_class(self):
    if self.action == "list":
        return ProductListSerializer
    return ProductDetailSerializer
```

This snippet belongs in a ViewSet class and relies on its `action` attribute. For custom actions, define and test the intended serializer explicitly.

# Advanced Example

Dynamic writable fields are especially risky: removing a field may silently ignore client input, while changing requiredness can weaken validation. Prefer separate input and output serializers when behavior differs.

# Under the Hood

Serializer fields are cloned/bound per serializer instance from declared fields. Instance-level modifications affect that serializer instance, not global class definitions, provided they occur after superclass initialization.

# Performance Notes

Dynamic field construction has modest overhead. The selected fields can significantly alter related-object queries and payload size; optimize queryset to match the chosen representation.

# Best Practices

- Use separate serializer classes for distinct contracts.
- Keep dynamic changes allowlisted and context-driven.
- Never make privileged fields user-selectable.
- Call `super().__init__` and mutate only instance fields.

# Common Mistakes

- Modifying class-level fields globally.
- Removing fields without documenting response variation.
- Trusting query parameters for authorization.
- Using one dynamic serializer for unrelated create/list/detail shapes.

# Debugging

Inspect `serializer.fields` after instantiation for each context/action. Test compact/full output and permissions independently.

# Interview Questions

1. How choose serializer per action? Override `get_serializer_class`.
2. Where should dynamic fields be changed? On the serializer instance after `super().__init__`.
3. Should query parameters control access to sensitive fields? No.
4. When use separate serializers? When input/output contracts differ substantially.

# Practice Exercises

1. Create compact list and detailed serializer classes.
2. Select them by ViewSet action.
3. Test that no dynamic option exposes server-owned/private fields.

# Chapter Summary

Dynamic serializers adapt representation to controlled context, but flexibility can obscure contracts. Prefer distinct serializers for meaningful differences and never let client field selection bypass permissions.

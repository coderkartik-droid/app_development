# Introduction

`SerializerMethodField` is a read-only serializer field whose value comes from a serializer method. It is useful for small computed values that do not map directly to a model field.

# Why this concept exists

Clients sometimes need a derived representation such as a display label or a simple count. A method field keeps that output near the serializer contract.

# Real-world analogy

It is a calculated column on a report: the value is derived when the report is prepared rather than stored as an independent fact.

# Internal Working

DRF calls `get_<field_name>(self, obj)` during serialization. The method's result must be renderable primitive data; it is read-only and does not accept client input.

# Request Lifecycle (where applicable)

```text
ORM instance -> serializer field iteration -> get_field(obj) -> primitive -> Response
```

# Syntax

```python
label = serializers.SerializerMethodField()

def get_label(self, obj):
    return obj.name.upper()
```

# Complete Example 1

Add a computed display label to a model representation. Assume `Product` has `name` and `price_cents`.

```python
# shop/serializers.py
from rest_framework import serializers
from .models import Product


class ProductSerializer(serializers.ModelSerializer):
    display_label = serializers.SerializerMethodField()

    class Meta:
        model = Product
        fields = ["id", "name", "price_cents", "display_label"]

    def get_display_label(self, obj):
        return f"{obj.name} — {obj.price_cents} cents"
```

```python
# shop/views.py
from rest_framework import generics
from .models import Product
from .serializers import ProductSerializer


class ProductListView(generics.ListAPIView):
    queryset = Product.objects.order_by("id")
    serializer_class = ProductSerializer
```

```python
# shop/urls.py
from django.urls import path
from .views import ProductListView

urlpatterns = [path("products/", ProductListView.as_view())]
```

**Line-by-line:** Declaring `SerializerMethodField` adds output field `display_label`. Meta allowlists it. DRF calls `get_display_label` with each product instance. The generic list view loads products and applies serializer; route exposes list.

**Request → Serializer → Validation → Database → Response:** GET -> queryset loads products -> serializer iterates output fields -> method computes label -> Response rendered. There is no input validation/write. **Internal working:** Method fields are read-only. **Professional use case:** Small derived labels or booleans useful to clients. Removing the matching method produces an attribute error during serialization; if the method accesses relations, not optimizing queries can cause N+1 behavior.

# Complete Example 2

Use serializer context for request-dependent output:

```python
def get_is_owner(self, obj):
    request = self.context.get("request")
    return bool(request and request.user.is_authenticated and obj.owner_id == request.user.id)
```

This exposes whether the current authenticated user owns the object. Generic DRF views usually include request in serializer context. The value is informational only; permissions must still protect mutations.

# Complete Example 3

For a count of prefetched related items:

```python
def get_item_count(self, obj):
    return len(obj.items.all())
```

This may use a prefetched cache when the queryset uses `prefetch_related("items")`; otherwise one query per object may occur. For database aggregation, annotate the queryset and read the annotation instead.

# Advanced Example

If computation is expensive or queried repeatedly, calculate it in a queryset annotation/service rather than a Python method per row. Method field is a representation hook, not a general business workflow.

# Under the Hood

`SerializerMethodField` defaults its method name to `get_<field_name>` and is read-only. Its representation calls that method for each object being serialized.

# Performance Notes

Methods run per serialized object. Avoid database lookups, remote calls, and expensive repeated calculations inside them.

# Best Practices

- Keep computed fields deterministic and cheap.
- Optimize relation access in the view's queryset.
- Add explicit type hints/documentation for complex output.
- Do not use output booleans as authorization controls.

# Common Mistakes

- Method name mismatch.
- Returning a model/queryset instead of primitive data.
- N+1 queries in the getter.
- Expecting clients to submit a method field.

# Debugging

Check the declared field name, corresponding method, object attribute, context presence, and query count. Inspect serializer output for each object type.

# Interview Questions

1. What is SerializerMethodField? Read-only computed representation field.
2. What method is called? `get_<field_name>(obj)` by default.
3. Is it writable? No.
4. What is its main performance risk? Per-object expensive work or queries.

# Practice Exercises

1. Add a normalized display name field.
2. Add an ownership indicator and test anonymous requests.
3. Count related items without N+1 queries.

# Chapter Summary

SerializerMethodField computes output by calling a method per object. Use it for cheap representation-only values, optimize related data access, and never confuse displayed indicators with access control.

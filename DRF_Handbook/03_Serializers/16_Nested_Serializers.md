# Introduction

A nested serializer represents a related object inside another serializer's output or input. Read-only nesting is straightforward; writable nesting requires explicit rules for creating, updating, matching, and deleting related records.

# Why this concept exists

Clients often need a convenient representation of related data. Nesting can reduce extra requests, but it changes payload shape and can increase query and write complexity.

# Real-world analogy

A purchase order may include its line items in one document. Showing them together is easy; editing the whole order requires rules for which lines are added, changed, or removed.

# Internal Working

```text
Parent instance -> nested field -> related instances -> primitive output
Nested input -> child serializer validation -> explicit parent create/update logic
```

DRF can represent nested relations by declaring a serializer field. Writable nested persistence is not inferred automatically for arbitrary relationships.

# Request Lifecycle (where applicable)

For output, the parent serializer accesses the relation and nested serializer converts it. For input, nested fields validate their own shape; parent `create`/`update` must implement relationship persistence or use an explicitly supported relation field.

# Syntax

```python
class ItemSerializer(serializers.ModelSerializer):
    class Meta:
        model = Item
        fields = ["id", "name"]

class OrderSerializer(serializers.ModelSerializer):
    items = ItemSerializer(many=True, read_only=True)
```

# Complete Example 1

Read-only order details nested with line items. Assume `Order` has reverse relation `items` to `OrderItem`.

```python
# orders/serializers.py
from rest_framework import serializers
from .models import Order, OrderItem


class OrderItemSerializer(serializers.ModelSerializer):
    class Meta:
        model = OrderItem
        fields = ["id", "product_name", "quantity"]
        read_only_fields = ["id"]


class OrderSerializer(serializers.ModelSerializer):
    items = OrderItemSerializer(many=True, read_only=True)

    class Meta:
        model = Order
        fields = ["id", "reference", "items"]
        read_only_fields = ["id"]
```

```python
# orders/views.py
from rest_framework import generics
from .models import Order
from .serializers import OrderSerializer


class OrderDetailView(generics.RetrieveAPIView):
    queryset = Order.objects.prefetch_related("items")
    serializer_class = OrderSerializer
```

```python
# orders/urls.py
from django.urls import path
from .views import OrderDetailView

urlpatterns = [path("orders/<int:pk>/", OrderDetailView.as_view())]
```

**Line-by-line:** Child serializer explicitly selects line item fields and protects its ID. Parent declares `items` as many related records and read-only. Parent Meta exposes reference and nested items. Queryset prefetches many-valued relation to avoid per-order queries. Retrieve view returns one order.

**Request → Serializer → Validation → Database → Response:** GET -> ORM retrieves order and prefetched items -> parent serializer invokes child serializer -> primitives -> renderer. No input validation/write occurs in this read-only example. **Internal working:** `many=True` creates a list representation. **Professional use case:** Order detail screen needing line items in one bounded response. Removing `prefetch_related` may cause N+1 queries on lists; removing `read_only=True` does not automatically implement safe nested writes.

# Complete Example 2

Explicitly create a parent and nested children:

```python
from django.db import transaction


class OrderWriteSerializer(serializers.ModelSerializer):
    items = OrderItemSerializer(many=True)

    class Meta:
        model = Order
        fields = ["reference", "items"]

    @transaction.atomic
    def create(self, validated_data):
        items_data = validated_data.pop("items")
        order = Order.objects.create(**validated_data)
        OrderItem.objects.bulk_create([
            OrderItem(order=order, **item_data)
            for item_data in items_data
        ])
        return order
```

This is a complete serializer class when related model fields align. The transaction ensures all-or-nothing creation. `bulk_create` may skip model save hooks/signals depending on backend; use individual creates if those hooks are required.

# Complete Example 3

For a simpler writable foreign key, prefer a related field:

```python
category = serializers.PrimaryKeyRelatedField(
    queryset=Category.objects.all()
)
```

The submitted primary key is resolved against the queryset and validated as a relation. Restrict the queryset to objects the caller may choose; an unrestricted valid relation can still be unauthorized.

# Advanced Example

Define nested update semantics explicitly: child IDs match existing rows; unknown IDs are rejected or created; omitted children are preserved or deleted according to contract; duplicate IDs are rejected. Wrap changes in a transaction and test each branch.

# Under the Hood

Nested output recursively invokes child representation. Nested input produces nested validated structures; DRF cannot infer an unambiguous multi-object write policy and therefore requires explicit persistence behavior for writable nested relations.

# Performance Notes

Nested payloads can balloon. Use `prefetch_related` for collections, pagination or a separate endpoint for large relations, and avoid deep nesting.

# Best Practices

- Prefer read-only nesting unless nested writes are truly needed.
- Optimize related queries alongside serializer design.
- Document nested shape and cardinality.
- Make nested create/update/delete semantics explicit and transactional.

# Common Mistakes

- Assuming writable nested data saves automatically.
- Omitting `many=True` for collections.
- N+1 database queries in nested output.
- Accepting related IDs without authorization scoping.

# Debugging

Inspect nested serializer errors, relation attribute name, `many` setting, query count, and overridden create/update. Test empty, omitted, duplicate, and unknown child IDs.

# Interview Questions

1. What does `many=True` mean? The nested relation contains multiple instances.
2. Does DRF automatically save nested writes? Generally no; implement explicit behavior.
3. Why prefetch related children? Reduce per-parent queries.
4. What policy must nested update define? Child identity and add/change/delete behavior.

# Practice Exercises

1. Serialize an order with read-only nested lines.
2. Optimize it with `prefetch_related`.
3. Implement transactional nested create and test rollback.

# Chapter Summary

Nested serializers simplify related representations but introduce query and write complexity. Read-only nesting is a safe starting point; writable nesting requires explicit identity, lifecycle, authorization, and transaction rules.

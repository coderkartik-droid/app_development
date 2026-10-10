# Introduction

Real-world serializers represent deliberate API contracts for common domains such as profiles, catalog items, and orders. These examples emphasize field direction, validation, and the separation between client input and server authority.

# Why this concept exists

A field list that looks convenient can expose confidential or editable values. Domain examples reveal how to make API boundaries intentional.

# Real-world analogy

Each serializer is a department-specific form: a public product card, an account profile, or an order request asks for and reveals different information.

# Internal Working

```text
request -> parser -> input serializer -> validation -> authorized service/ORM
        -> output serializer -> Response -> renderer
```

Input and output shapes may differ. The same model can have multiple serializers for list, detail, creation, and privileged staff operations.

# Request Lifecycle (where applicable)

The view applies authentication and permissions first, serializer validates input, trusted server values are injected, a transaction persists related changes, and an output serializer returns approved fields.

# Syntax

```python
class ProductCardSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ["id", "name", "price_cents"]
```

# Complete Example 1

Public product catalog representation with stock count. Assumes `Product` has `name`, `price_cents`, `is_published`, and a `stock_entries` relation with `quantity`.

```python
# shop/serializers.py
from rest_framework import serializers
from .models import Product


class ProductCardSerializer(serializers.ModelSerializer):
    available_quantity = serializers.IntegerField(read_only=True)

    class Meta:
        model = Product
        fields = ["id", "name", "price_cents", "available_quantity"]
        read_only_fields = fields
```

```python
# shop/views.py
from django.db.models import Sum
from django.db.models.functions import Coalesce
from rest_framework import generics
from .models import Product
from .serializers import ProductCardSerializer


class PublishedProductList(generics.ListAPIView):
    serializer_class = ProductCardSerializer

    def get_queryset(self):
        return (
            Product.objects.filter(is_published=True)
            .annotate(
                available_quantity=Coalesce(
                    Sum("stock_entries__quantity"), 0
                )
            )
            .order_by("id")
        )
```

```python
# shop/urls.py
from django.urls import path
from .views import PublishedProductList

urlpatterns = [path("products/", PublishedProductList.as_view())]
```

**Line-by-line:** `available_quantity` is read-only because it is an annotation, not client input. Meta allowlists public fields. View filters unpublished products, aggregates stock in SQL, coalesces missing totals to zero, and orders results. **Request → Serializer → Validation → Database → Response:** GET -> scoped queryset/aggregate -> serializer represents annotated rows -> response renderer. No input validation/write occurs. **Internal working:** Annotation prevents one stock query per product. **Professional use case:** Public catalog cards. Removing publication filtering can expose draft products; exposing internal cost fields would violate the field allowlist.

# Complete Example 2

Profile write contract:

```python
class ProfileWriteSerializer(serializers.ModelSerializer):
    class Meta:
        model = Profile
        fields = ["display_name", "bio"]
```

Only user-editable fields are accepted. The view must select the profile belonging to the authenticated user; serializer field restriction alone does not guarantee that.

# Complete Example 3

Order line input:

```python
class OrderLineInputSerializer(serializers.Serializer):
    product_id = serializers.IntegerField(min_value=1)
    quantity = serializers.IntegerField(min_value=1, max_value=20)
```

This validates shape/range but not stock, price, or authorization. The service must look up current product and inventory and compute authoritative totals server-side.

# Advanced Example

For checkout, accept product identifiers and quantities only. In one transaction, re-read current prices/stock, validate availability, create order and lines, decrement inventory, and store the server-calculated total. Use concurrency controls appropriate to the database; never trust a submitted total.

# Under the Hood

Serializer fields define wire representation; queryset annotations/related loading supply efficient data. Views/services enforce access and invariants; database constraints backstop concurrent changes.

# Performance Notes

Aggregate in SQL, paginate catalog lists, and avoid nested line-item queries in list serializers. Profile query counts and response payload size.

# Best Practices

- Separate read and write serializers when shapes differ.
- Keep price, stock, ownership, and permissions server-controlled.
- Explicitly allowlist fields.
- Use transactions for order and inventory mutations.

# Common Mistakes

- Accepting client totals or stock quantities.
- Using a public catalog serializer for administrative writes.
- Serializing unfiltered unpublished records.
- Querying related stock separately for every product.

# Debugging

Inspect fields, queryset filters/annotations, permission outcomes, actual SQL query count, and transaction results. Test cross-user and stale-stock cases.

# Interview Questions

1. Why separate read/write serializers? Public output and accepted input often differ.
2. Where calculate checkout total? On the server from authoritative current data.
3. Why annotate stock? Avoid per-product queries.
4. What remains outside serializer validation? Authorization, concurrency, and complete business transaction policy.

# Practice Exercises

1. Implement the product card with a stock annotation.
2. Create a profile update endpoint scoped to current user.
3. Design transactional order creation accepting IDs/quantities only.

# Chapter Summary

Real-world serializer design starts from what clients may send and see. Keep server authority over ownership, pricing, and inventory; optimize representation queries and make multi-model writes transactional.

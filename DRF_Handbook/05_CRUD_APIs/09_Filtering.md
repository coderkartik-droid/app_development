# Introduction

Filtering narrows a collection response according to client-supplied criteria, such as category, active state, or a price range.

# Why this concept exists

Returning every row and forcing the client to filter wastes bandwidth, memory, and time. Server-side filtering lets the database use indexes and returns only relevant records.

# Real-world analogy

Filtering a product catalogue is like asking a shopkeeper for “active lamps under $30” rather than carrying the whole inventory home.

# Internal Working

A filter backend or `get_queryset()` receives validated request parameters and composes Django QuerySet conditions. SQL executes when pagination or serialization evaluates the queryset.

# Request Lifecycle

Request query -> validate filter values -> apply ORM predicates -> paginate -> serialize matching rows -> response.

# Syntax

```python
queryset.filter(category_id=category_id, price__lte=max_price)
```

# Complete Example 1

```python
# settings.py
REST_FRAMEWORK = {
    "DEFAULT_FILTER_BACKENDS": [
        "django_filters.rest_framework.DjangoFilterBackend",
    ],
}
```

```python
# views.py
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.generics import ListAPIView
from .models import Product
from .serializers import ProductSerializer

class ProductListView(ListAPIView):
    queryset = Product.objects.filter(active=True)
    serializer_class = ProductSerializer
    filter_backends = [DjangoFilterBackend]
    filterset_fields = ["category", "active"]
```

Install `django-filter` and add `django_filters` to `INSTALLED_APPS` for this implementation. The backend translates allowlisted fields into ORM filters; the base queryset still hides inactive records, so adjust intentionally if clients may request them. Do not expose sensitive model fields as filters.

# Complete Example 2

```python
from decimal import Decimal, InvalidOperation
from rest_framework.exceptions import ValidationError

def filter_by_max_price(queryset, raw_value):
    if raw_value is None:
        return queryset
    try:
        maximum = Decimal(raw_value)
    except InvalidOperation as exc:
        raise ValidationError({"max_price": "Enter a decimal value."}) from exc
    if maximum < 0:
        raise ValidationError({"max_price": "Must be non-negative."})
    return queryset.filter(price__lte=maximum)
```

The function validates conversion and range before using a parameterized ORM filter. It avoids raw SQL interpolation. Call it from `get_queryset`; use `Decimal` for money rather than binary floating point.

# Complete Example 3

```python
response = APIClient().get("/api/products/?category=3&active=true")
assert response.status_code == 200
```

This request demonstrates multiple filters. A complete test should seed matching and nonmatching products, then assert the exact returned IDs and reject malformed parameters.

# Advanced Example

Define a `FilterSet` for range lookups and custom filters; combine it with pagination. Ensure indexes and query plans support frequent filter combinations.

# Under the Hood

Filter backends run as part of generic list processing before pagination/serialization. QuerySet filters are immutable expressions that return new lazy querysets; values are bound as SQL parameters.

# Performance Notes

Index commonly filtered columns, avoid unrestricted joins, limit filter complexity, and inspect `EXPLAIN` plans for expensive combinations.

# Best Practices

- Allowlist fields and lookup types.
- Validate numeric/date/boolean inputs and define empty-value behavior.
- Combine filtering with stable ordering and pagination.
- Keep tenant scope in the base queryset, not a client-controlled filter.

# Common Mistakes

- Letting filters replace authorization scope.
- Using unvalidated string coercion.
- Creating an index for every hypothetical filter rather than measured usage.

# Debugging

Inspect `request.query_params`, filter backend configuration, SQL, result count, and expected access scope. Test missing, invalid, boundary, and combined values.

# Interview Questions

1. Where do DRF filter backends run? In generic view queryset filtering.
2. Why keep authorization scope separate? Clients must not widen visibility.
3. Why use ORM filtering? It safely parameterizes values and composes SQL.

# Practice Exercises

1. Add category and maximum-price filters.
2. Define allowed filtering for a public and private product list.
3. Test invalid decimal and tenant-isolation inputs.

# Chapter Summary

Filtering belongs in the database query path, but its inputs must be constrained. Keep security scope fixed, expose only deliberate filters, and measure query plans.

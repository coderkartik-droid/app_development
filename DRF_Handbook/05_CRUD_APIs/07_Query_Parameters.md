# Introduction

Query parameters are key-value pairs after `?` in a URL. They commonly control filtering, pagination, search, and ordering without changing which resource path is addressed.

# Why this concept exists

A client can request a useful slice of a collection while keeping one stable endpoint, such as `/products/?active=true&page=2`.

# Real-world analogy

The path is a catalogue shelf; query parameters are the instructions to show only active items, sorted by price, on page two.

# Internal Working

DRF exposes parsed query values as `request.query_params` (an alias for Django's `GET` QueryDict). Views or filter backends translate these untrusted strings into validated queryset operations.

# Request Lifecycle

URL resolver selects the collection view; query string remains request metadata; view/filter backend validates and applies options; serializer returns the resulting page.

# Syntax

```python
status_value = request.query_params.get("status")
```

# Complete Example 1

```python
from rest_framework import generics
from rest_framework.exceptions import ValidationError
from .models import Product
from .serializers import ProductSerializer

class ProductListView(generics.ListAPIView):
    serializer_class = ProductSerializer

    def get_queryset(self):
        queryset = Product.objects.all()
        active = self.request.query_params.get("active")
        if active is not None:
            if active not in {"true", "false"}:
                raise ValidationError({"active": "Use true or false."})
            queryset = queryset.filter(active=(active == "true"))
        return queryset.order_by("id")
```

The method starts with a queryset, reads a string, validates its accepted values, converts it to a boolean, and returns stable ordering. Query parameters are untrusted input; direct truthiness would treat `"false"` as true. Add pagination to avoid an unbounded result set.

# Complete Example 2

```python
from rest_framework.test import APIClient

response = APIClient().get("/api/products/?active=false")
assert response.status_code == 200
assert all(item["active"] is False for item in response.data["results"])
```

The request encodes a boolean as text; the view performs conversion. This assertion assumes pagination is configured and response data uses a `results` key. Without pagination, assert directly against the returned list instead.

# Complete Example 3

```python
from django.http import QueryDict

query = QueryDict("tag=blue&tag=clearance")
tags = query.getlist("tag")
assert tags == ["blue", "clearance"]
```

QueryDict supports repeated keys; `.get()` returns one value while `.getlist()` preserves all. Use repeated values for documented multi-select filters and cap their count.

# Advanced Example

Use DRF filter backends for reusable filtering/search/order behavior. Whitelist fields and validate types/ranges; never interpolate query strings into raw SQL.

# Under the Hood

Django parses the URL query into an immutable-by-default `QueryDict`. QuerySets remain lazy as filters are composed; SQL runs when results are evaluated.

# Performance Notes

Index common filters, bound page sizes, cap multi-value filters, and avoid expensive joins across unconstrained user input.

# Best Practices

- Publish accepted names, types, defaults, and error behavior.
- Use `request.query_params`, not `request.data`, for URL query values.
- Validate conversions and allowlist ordering/filter fields.

# Common Mistakes

- Treating string `"false"` as a truthy boolean.
- Assuming query values are integers or safe SQL fragments.
- Ignoring repeated keys or excessive parameter counts.

# Debugging

Print/log sanitized `request.query_params`, inspect generated SQL, and test empty, malformed, repeated, and boundary values.

# Interview Questions

1. Where should GET filters be read? `request.query_params`.
2. Why is a QueryDict special? It supports repeated keys.
3. Are query parameters trusted? No; validate and constrain them.

# Practice Exercises

1. Add a bounded `min_price`/`max_price` filter.
2. Support multiple category IDs safely.
3. Write tests for malformed booleans and ranges.

# Chapter Summary

Query parameters customize collection requests. Parse their string representation deliberately, whitelist supported behavior, and keep query cost and output size bounded.

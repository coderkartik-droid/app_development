# Introduction

Searching matches user-entered text against selected fields, such as product name, SKU, or description. DRF includes a `SearchFilter` backend for common search behavior.

# Why this concept exists

Users often know a word or phrase but not an exact ID. Search makes large collections discoverable with a simple request.

# Real-world analogy

Search is asking a librarian to find books mentioning a phrase, rather than browsing every shelf.

# Internal Working

The search backend reads the configured query parameter, creates ORM lookups over allowlisted `search_fields`, combines conditions, and filters the queryset before pagination.

# Request Lifecycle

`?search=lamp` -> SearchFilter -> ORM text predicates -> paginated result -> serialized response.

# Syntax

```python
filter_backends = [SearchFilter]
search_fields = ["name", "sku", "description"]
```

# Complete Example 1

```python
from rest_framework import filters, generics
from .models import Product
from .serializers import ProductSerializer

class ProductSearchView(generics.ListAPIView):
    queryset = Product.objects.filter(active=True)
    serializer_class = ProductSerializer
    filter_backends = [filters.SearchFilter]
    search_fields = ["name", "sku", "description"]
```

`SearchFilter` reads the default `search` parameter. Fields are explicitly allowlisted, and active filtering remains part of the base queryset. The default matching may use case-insensitive containment and can be costly on large tables; database-specific full-text search may be preferable at scale.

# Complete Example 2

```python
from rest_framework.test import APIClient

response = APIClient().get("/api/products/?search=lamp")
assert response.status_code == 200
```

The request reaches the configured backend before pagination. A useful test creates one matching and one nonmatching object and asserts only the correct one appears. Escape/encoding is handled by the HTTP client; cap search length to reduce abuse.

# Complete Example 3

```python
class ProductSearchView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [filters.SearchFilter]
    search_fields = ["^sku", "=barcode", "name"]
```

DRF prefixes adjust lookup behavior: `^` is starts-with and `=` is exact matching. This can improve precision for identifiers. Confirm generated SQL and use a database index compatible with the chosen lookup.

# Advanced Example

For multilingual or relevance-ranked search, use a dedicated full-text search facility or search service, while preserving tenant filtering and pagination at the API boundary.

# Under the Hood

SearchFilter parses terms and combines field lookups; multiple terms are generally treated as an AND across term groups, while fields form alternatives for each term. Exact behavior should be verified against the installed DRF version.

# Performance Notes

Leading-wildcard contains searches often cannot use ordinary B-tree indexes. Limit fields and query length, paginate results, and move to full-text indexes when measured load justifies it.

# Best Practices

- Search only fields users should discover.
- Keep visibility restrictions in the queryset.
- Document tokenization and matching behavior.
- Consider rate limits and minimum query length for expensive search.

# Common Mistakes

- Searching passwords, private notes, or hidden attributes.
- Assuming substring search scales indefinitely.
- Forgetting tenant scoping when adding a filter backend.

# Debugging

Inspect `search_fields`, the `search` parameter, generated SQL, collation/case behavior, and result pagination.

# Interview Questions

1. Which DRF backend provides basic search? `SearchFilter`.
2. Why configure fields explicitly? Control semantics and prevent data exposure/cost.
3. When use full-text search? At scale or when tokenization/ranking/language support is needed.

# Practice Exercises

1. Search by product name and exact SKU.
2. Test matching, case behavior, and empty queries.
3. Compare query plans for contains versus prefix lookups.

# Chapter Summary

Basic DRF search is convenient for modest datasets. Explicitly choose searchable fields, retain access scoping, and replace broad substring scans with indexed search when usage grows.

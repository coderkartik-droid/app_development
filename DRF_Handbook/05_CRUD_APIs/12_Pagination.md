# Introduction

Pagination divides a large collection into bounded responses. DRF offers page-number, limit/offset, and cursor approaches.

# Why this concept exists

Without pagination, a list endpoint can transfer and serialize an unbounded number of rows, degrading server and client performance.

# Real-world analogy

Pagination is reading a book one chapter at a time instead of photocopying the entire library catalogue in one request.

# Internal Working

The paginator receives a queryset and request parameters, computes a slice and navigation metadata, then the serializer processes only the selected page. Some styles perform a total count query.

# Request Lifecycle

List queryset -> filtering/ordering -> paginator selects page -> serializer transforms page -> response includes results and navigation metadata.

# Syntax

```python
REST_FRAMEWORK = {"DEFAULT_PAGINATION_CLASS": "rest_framework.pagination.PageNumberPagination",
                  "PAGE_SIZE": 25}
```

# Complete Example 1

```python
# settings.py
REST_FRAMEWORK = {
    "DEFAULT_PAGINATION_CLASS": "rest_framework.pagination.PageNumberPagination",
    "PAGE_SIZE": 25,
}
```

This globally enables page-number pagination with 25 records per page. Generic list views apply it automatically. The response includes `count`, `next`, `previous`, and `results`. Without pagination, a plain array is returned and growth is unbounded.

# Complete Example 2

```python
from rest_framework.pagination import PageNumberPagination
from rest_framework.generics import ListAPIView

class ProductPagination(PageNumberPagination):
    page_size = 20
    page_size_query_param = "page_size"
    max_page_size = 100

class ProductListView(ListAPIView):
    queryset = Product.objects.order_by("pk")
    serializer_class = ProductSerializer
    pagination_class = ProductPagination
```

The client can request a smaller or larger page, but `max_page_size` caps resource use. Stable ordering is important. This configuration belongs to an app with the referenced model and serializer installed.

# Complete Example 3

```python
response = APIClient().get("/api/products/?page=2&page_size=10")
assert response.status_code == 200
assert len(response.data["results"]) <= 10
assert "next" in response.data and "previous" in response.data
```

The test asserts the pagination envelope and bound, not exact fixture content. A complete fixture can verify navigation and page membership. For an out-of-range page, document whether the endpoint returns 404 or a customized empty page.

# Advanced Example

Cursor pagination is useful for large, frequently changing feeds. It requires a consistent ordering field and avoids expensive large offsets, but clients navigate with opaque cursors rather than arbitrary page numbers.

# Under the Hood

Page-number pagination generally counts rows and slices by offset. Limit/offset gives clients direct control and therefore must cap the limit. Cursor pagination encodes position and ordering state.

# Performance Notes

Large offset scans can become slow; total count can also be expensive. Choose pagination style based on collection size, update rate, and client navigation needs.

# Best Practices

- Always bound collection responses.
- Cap client-controlled page sizes.
- Use stable ordering and test boundary pages.
- Treat cursors as opaque tokens.

# Common Mistakes

- Unbounded page-size query parameters.
- Missing ordering, resulting in unstable pages.
- Changing response from list to paginated object without versioning/documentation.

# Debugging

Check global/view pagination class, page parameter spelling, count/query cost, ordering, and response envelope.

# Interview Questions

1. What are common DRF pagination styles? Page number, limit/offset, cursor.
2. Which is suitable for a changing feed? Often cursor pagination.
3. Why cap page size? Resource protection.

# Practice Exercises

1. Add page-number pagination with a maximum size.
2. Write tests for first, middle, and invalid pages.
3. Compare offset and cursor behavior for a growing feed.

# Chapter Summary

Pagination bounds work and shapes the collection response. Select the strategy deliberately, enforce limits, and use deterministic ordering.

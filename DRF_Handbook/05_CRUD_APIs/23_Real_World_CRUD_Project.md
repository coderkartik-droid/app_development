# Introduction

This project builds a small product catalogue API with creation, listing, retrieval, update, deletion, filtering, search, ordering, and pagination.

# Why this concept exists

A coherent project shows how individual DRF features combine into one usable and maintainable resource API.

# Real-world analogy

The API is the digital inventory desk for a small shop: staff manage stock records while customers browse only active items.

# Internal Working

```text
client -> router -> auth/permissions -> viewset -> serializer
       -> scoped queryset/model -> database -> response renderer
```

# Request Lifecycle

Reads use a filtered and paginated queryset; writes validate input, enforce ownership and constraints, persist changes, and return standard status codes.

# Syntax

```http
GET /api/v1/products/?search=lamp&ordering=-price&page=1
```

# Complete Example 1

```python
# models.py
from django.conf import settings
from django.db import models

class Product(models.Model):
    owner = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    name = models.CharField(max_length=120)
    sku = models.CharField(max_length=40, unique=True)
    price = models.DecimalField(max_digits=9, decimal_places=2)
    active = models.BooleanField(default=True)

    class Meta:
        ordering = ["name", "pk"]
        indexes = [models.Index(fields=["active", "name"])]
```

```python
# serializers.py
from rest_framework import serializers
from .models import Product

class ProductSerializer(serializers.ModelSerializer):
    class Meta:
        model = Product
        fields = ["id", "name", "sku", "price", "active"]
        read_only_fields = ["id"]

    def validate_price(self, value):
        if value < 0:
            raise serializers.ValidationError("Price cannot be negative.")
        return value
```

```python
# views.py and urls.py
from rest_framework import filters, viewsets
from rest_framework.permissions import IsAuthenticated
from rest_framework.pagination import PageNumberPagination
from rest_framework.routers import SimpleRouter
from .models import Product
from .serializers import ProductSerializer

class ProductPagination(PageNumberPagination):
    page_size = 20
    page_size_query_param = "page_size"
    max_page_size = 100

class ProductViewSet(viewsets.ModelViewSet):
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticated]
    pagination_class = ProductPagination
    filter_backends = [filters.SearchFilter, filters.OrderingFilter]
    search_fields = ["name", "sku"]
    ordering_fields = ["name", "price"]

    def get_queryset(self):
        return Product.objects.filter(owner=self.request.user, active=True)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)

router = SimpleRouter()
router.register("products", ProductViewSet, basename="product")
urlpatterns = router.urls
```

**Line-by-line:** The model stores owner and catalogue values; SKU is unique and the active/name index supports a common access path. The serializer validates price and keeps ID read-only. Pagination caps page size. Search/order backends expose only chosen fields. The viewset supplies standard CRUD actions; `get_queryset()` scopes access and `perform_create()` assigns trusted ownership. Apply model migrations and include router URLs under the versioned API prefix before running.

**Flow:** POST -> router create -> authenticated policy -> input validation -> save with owner -> database insert -> 201. GET -> owner/active queryset -> search/order -> page -> serializer -> response. Detail updates/deletes use the same scoped queryset. If owner scoping is removed, users can access other users' products; if `active=True` is applied globally, owners cannot retrieve inactive records—split public and management policies when needed.

# Complete Example 2

```python
# Include in project urls.py
from django.urls import include, path

urlpatterns = [path("api/v1/", include("catalog.urls"))]
```

The include mounts router-generated paths below `/api/v1/`. The version segment creates a clear compatibility boundary. Ensure the app is installed, migrations applied, and authentication configured. Without this include, generated routes are unreachable.

# Complete Example 3

```python
response = APIClient().post(
    "/api/v1/products/",
    {"name": "Desk lamp", "sku": "LAMP-01", "price": "24.90", "active": True},
    format="json",
)
assert response.status_code == 201
assert response.data["sku"] == "LAMP-01"
```

Authenticate the test client before sending this request; create a test user and assert the saved product owner is that user. Add tests for duplicate SKU (400), negative price (400), cross-user detail (404), and pagination.

# Advanced Example

For production, add explicit object policy, audit fields, schema docs, throttling, transaction-aware domain operations, structured logging, and integration tests. Decide whether inactive products should be hidden from owners or only public catalogue views.

# Under the Hood

Router maps HTTP verbs to ModelViewSet actions. Generic mixins call `get_queryset`, instantiate serializers, and use ORM operations. Response rendering happens after the action returns.

# Performance Notes

Use pagination and indexes based on observed filters; add `select_related`/`prefetch_related` if serializer grows related fields. Measure query counts with realistic data.

# Best Practices

- Keep owner assignment server-side.
- Separate public visibility from management access where requirements differ.
- Apply database constraints and API validation.
- Document filtering, sorting, page limits, and errors.

# Common Mistakes

- Assuming authentication alone scopes product IDs.
- Making SKU uniqueness checks only in Python.
- Forgetting migration and URL include steps.

# Debugging

Verify router URLs, authentication, active/owner filters, serializer errors, migration state, and SQL query count. Use tests for each permission boundary.

# Interview Questions

1. Why override `get_queryset()`? Request-specific visibility and filtering.
2. What does the router map? URL and HTTP method to ViewSet action.
3. Why both SKU validation and a unique constraint? Helpful errors and concurrency-safe integrity.

# Practice Exercises

1. Add categories and filter by category.
2. Add soft-delete and owner management visibility.
3. Document the API schema and run contract tests.

# Chapter Summary

The project combines routing, authentication, serializer validation, scoped querysets, CRUD actions, filtering, ordering, pagination, and database constraints into one coherent API.

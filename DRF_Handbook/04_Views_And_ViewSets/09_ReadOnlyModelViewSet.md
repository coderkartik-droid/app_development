# Introduction

`ReadOnlyModelViewSet` provides only `list` and `retrieve` actions for a queryset and serializer. It is useful for resources clients may inspect but not modify through that API.

# Why this concept exists

Read access and write access often have different policies. A read-only abstraction avoids exposing create/update/delete handlers accidentally.

# Real-world analogy

A public catalog is a display shelf: visitors can browse and inspect items, but only authorized staff can alter stock.

# Internal Working

```text
router -> list/retrieve action -> queryset -> serializer -> Response
```

The class combines generic viewset mechanics with read-only mixins; router maps collection GET to list and detail GET to retrieve.

# Request Lifecycle

Router creates only read action mappings. DRF dispatch and policy checks run, then the action queries and serializes. Write methods without mapped handlers return 405.

# Syntax

```python
class PublicProductViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Product.objects.filter(is_published=True)
    serializer_class = ProductSerializer
```

# Complete Example 1

Expose only published products. Assume `ProductSerializer` and model exist.

```python
# catalog/views.py
from rest_framework import viewsets
from rest_framework.permissions import AllowAny
from .models import Product
from .serializers import ProductSerializer


class PublicProductViewSet(viewsets.ReadOnlyModelViewSet):
    serializer_class = ProductSerializer
    permission_classes = [AllowAny]

    def get_queryset(self):
        return Product.objects.filter(is_published=True).order_by("name")
```

```python
# catalog/urls.py
from rest_framework.routers import SimpleRouter
from .views import PublicProductViewSet

router = SimpleRouter()
router.register("products", PublicProductViewSet, basename="public-product")
urlpatterns = router.urls
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("catalog.urls"))]
```

**Line-by-line:** ReadOnlyModelViewSet supplies list/retrieve only. `AllowAny` makes public access explicit. `get_queryset` filters unpublished products and orders results. Router maps standard read routes; project includes them.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** GET collection/detail -> action -> public policy -> filtered queryset/object -> serializer -> rendered response. No write/validation occurs. **Internal execution:** PUT/POST/DELETE have no action mapping and return 405. **Professional use case:** Public product catalog. Removing `is_published=True` may leak drafts; AllowAny should be deliberate, not copied to private data.

# Complete Example 2

Use a bounded/paginated read viewset:

```python
class PublicProductViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Product.objects.filter(is_published=True).order_by("id")
    serializer_class = ProductSerializer
    pagination_class = ProductPagination
```

`ProductPagination` must be a defined DRF pagination class. Pagination limits response and database work; ordering should be deterministic to avoid unstable pages.

# Complete Example 3

Test write rejection:

```python
response = self.client.post(
    reverse("public-product-list"),
    {"name": "Injected"},
    format="json",
)
self.assertEqual(response.status_code, 405)
```

The test verifies no create action is exposed. Also assert the database did not change.

# Advanced Example

Read-only at the view level does not mean data is immutable everywhere. Admin, other API routes, or jobs may still write. Use a separate staff write endpoint with appropriate permissions and audit rules.

# Under the Hood

ReadOnlyModelViewSet combines GenericViewSet with list/retrieve mixins. Routers inspect these actions and generate only corresponding standard method mappings.

# Performance Notes

Use pagination, select related objects required by the serializer, and apply database indexes for measured filters/orderings.

# Best Practices

- Filter visibility in `get_queryset`.
- Make public access explicit.
- Return only safe fields.
- Test both read success and write-method rejection.

# Common Mistakes

- Assuming read-only class filters records automatically.
- Exposing unpublished/private records in queryset.
- Returning huge unpaginated collections.
- Believing no write routes means data cannot change elsewhere.

# Debugging

Inspect generated routes, list/detail queryset, permissions, serializer fields, pagination, and 405 behavior.

# Interview Questions

1. What actions does ReadOnlyModelViewSet provide? List and retrieve.
2. Does it automatically make resources public? No.
3. How exclude unpublished records? Filter queryset.
4. How prove writes are unavailable? Test unsupported methods return 405.

# Practice Exercises

1. Expose a public read-only category catalog.
2. Filter archived records and add pagination.
3. Test list, detail, missing detail, and POST rejection.

# Chapter Summary

ReadOnlyModelViewSet provides list/retrieve without write actions. Secure visibility through queryset and policy, paginate, and test that mutations are not exposed.

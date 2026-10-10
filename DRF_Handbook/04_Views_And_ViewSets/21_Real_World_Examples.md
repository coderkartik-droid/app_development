# Introduction

This chapter applies DRF view patterns to common product requirements: public catalog reads, private user-owned resources, and custom domain actions.

# Why this concept exists

Production APIs combine multiple view techniques. Realistic examples show where abstraction, authorization, serialization, and query design meet.

# Real-world analogy

A business has a public showroom, a private customer account desk, and a specialized service counter; each needs distinct access and operations.

# Internal Working

```text
router/view -> policy -> scoped queryset -> serializer/service -> ORM transaction -> Response
```

# Request Lifecycle

Each request follows the same DRF dispatch lifecycle, but public and private endpoints have different permission and queryset scope.

# Syntax

```python
class CatalogViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Product.objects.filter(is_published=True)
    serializer_class = ProductSerializer
```

# Complete Example 1

Public read-only catalog. Assume a `Product` model and `ProductSerializer`.

```python
# catalog/views.py
from rest_framework import viewsets
from rest_framework.permissions import AllowAny
from .models import Product
from .serializers import ProductSerializer


class CatalogViewSet(viewsets.ReadOnlyModelViewSet):
    serializer_class = ProductSerializer
    permission_classes = [AllowAny]

    def get_queryset(self):
        return Product.objects.filter(is_published=True).order_by("name")
```

```python
# catalog/urls.py
from rest_framework.routers import SimpleRouter
from .views import CatalogViewSet

router = SimpleRouter()
router.register("catalog", CatalogViewSet, basename="catalog")
urlpatterns = router.urls
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("catalog.urls"))]
```

**Line-by-line:** ReadOnlyModelViewSet exposes list/detail only. `AllowAny` makes public access deliberate. `get_queryset` excludes drafts and orders consistently. Router maps read routes; project mounts them.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** GET -> router list/retrieve action -> public permission -> published queryset/object -> serializer -> rendered response. **Internal working:** Write methods are not mapped. **Professional use case:** Public product browsing. Removing publication filter exposes drafts; public access is not inherited from router.

# Complete Example 2

Private saved addresses:

```python
class AddressViewSet(viewsets.ModelViewSet):
    serializer_class = AddressSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Address.objects.filter(owner=self.request.user)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

The list/detail routes are scoped; creation derives owner. Address data is sensitive and should be minimized, protected, and audited as product requirements dictate.

# Complete Example 3

Custom transition action:

```python
@action(detail=True, methods=["post"])
def archive(self, request, pk=None):
    record = self.get_object()
    record.archive()
    return Response(self.get_serializer(record).data)
```

This should validate transition eligibility and persist atomically where needed. `get_object()` uses the viewset queryset and object policy; calling an unrestricted manager directly could bypass scoping.

# Advanced Example

For checkout, use a command serializer and domain service; the view handles identity/policy and returns a resource result. The service calculates authoritative price, checks stock, and uses a transaction. Avoid embedding financial business workflows as serializer field callbacks.

# Under the Hood

ViewSet action routing, permission evaluation, queryset filtering, serializer invocation, ORM changes, and response rendering are separate stages. Hooks let each concern remain testable.

# Performance Notes

Catalog lists should paginate and optimize relations. Private lists should combine user filtering with indexes for common sort/filter keys.

# Best Practices

- Use read-only viewsets for public browse-only resources.
- Scope private records in `get_queryset`.
- Derive identity/authority server-side.
- Put complex state transitions in a transaction-aware service.

# Common Mistakes

- One ViewSet with overly broad permissions for all actions.
- Trusting client-submitted owner, total, or stock.
- Bypassing `get_object` in custom actions.
- Returning unbounded nested catalog data.

# Debugging

Test public/private identities, inspect queryset SQL, route action, serializer fields, and transaction effects. Reproduce stale-state/concurrency cases.

# Interview Questions

1. Why separate public/read-only and private CRUD views? Distinct actions and policies.
2. Where enforce record ownership? Queryset scoping plus appropriate object-level policy.
3. What does a custom action add? Resource-specific route/action beyond CRUD.
4. Where calculate authoritative order totals? Server-side service/domain logic.

# Practice Exercises

1. Build public catalog list/detail and hide drafts.
2. Build current-user address CRUD and test isolation.
3. Add transactional archive or publish action.

# Chapter Summary

Real APIs combine routing, policies, serializers, and querysets according to resource intent. Keep public/private boundaries explicit and place complex invariants in tested domain operations.

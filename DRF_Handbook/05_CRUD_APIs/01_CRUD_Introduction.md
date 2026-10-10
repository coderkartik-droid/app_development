# Introduction

CRUD means Create, Read, Update, and Delete: the four operations used to manage a resource. In a REST API these operations are commonly mapped to HTTP `POST`, `GET`, `PUT`/`PATCH`, and `DELETE`. This chapter builds a mental model for a complete DRF resource API.

# Why this concept exists

Applications need consistent ways to add, inspect, change, and remove records. CRUD endpoints make that behavior predictable for clients, while serializers validate the boundary and Django models/database constraints protect persisted data.

# Real-world analogy

Think of a product catalogue at a shop. Staff can add a product, browse products, correct details, or discontinue an item. Customers should only see permitted products; they should not be able to change internal stock fields arbitrarily.

# Internal Working

```text
HTTP request -> URL/router -> DRF view/action -> authentication/permissions
             -> serializer validation -> model/ORM -> database
             -> serialized representation -> HTTP response
```

The HTTP method expresses intent, not authorization. A `DELETE` route still needs permission checks, ownership rules, and a deliberate deletion policy.

# Request Lifecycle

DRF resolves the URL, initializes a `Request`, applies authentication and permissions, parses incoming content when needed, calls the selected action, validates data with a serializer, performs ORM work, and finalizes a `Response`. Invalid input normally becomes `400`; missing or inaccessible objects commonly become `404`.

# Syntax

| Intent | Method | Typical route | Typical success |
|---|---|---|---|
| Create | POST | `/api/products/` | 201 Created |
| List/read | GET | `/api/products/`, `/api/products/7/` | 200 OK |
| Replace | PUT | `/api/products/7/` | 200 or 204 |
| Partial update | PATCH | `/api/products/7/` | 200 or 204 |
| Delete | DELETE | `/api/products/7/` | 204 No Content |

# Complete Example 1

The following is a compact complete app-level implementation. It assumes a configured Django project with `rest_framework` and this app installed, and an applied migration for the model shown.

```python
# models.py
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=120)
    sku = models.CharField(max_length=40, unique=True)
    price = models.DecimalField(max_digits=9, decimal_places=2)
    active = models.BooleanField(default=True)
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
```

```python
# views.py and urls.py
from rest_framework import viewsets
from rest_framework.routers import SimpleRouter
from .models import Product
from .serializers import ProductSerializer

class ProductViewSet(viewsets.ModelViewSet):
    queryset = Product.objects.order_by("id")
    serializer_class = ProductSerializer

router = SimpleRouter()
router.register("products", ProductViewSet, basename="product")
urlpatterns = router.urls
```

**Line-by-line:** The model declares persisted columns and a uniqueness constraint. `ModelSerializer` derives fields and default create/update behavior from the model. `ModelViewSet` supplies list, retrieve, create, update, partial update, and destroy actions. `SimpleRouter` maps collection/detail URLs and HTTP methods to those actions.

**Flow:** `POST /products/` → router → `create` → serializer validation → ORM insert → database → serialized 201 response. A read follows the same path without the write. The view is intentionally open only for a teaching baseline; production must set authentication and permissions. Removing `unique=True` permits duplicate SKUs; removing serializer validation exposes database errors to users unless handled.

# Complete Example 2

Create a record directly through the serializer:

```python
serializer = ProductSerializer(data={"name": "Desk lamp", "sku": "LAMP-01", "price": "24.90", "active": True})
if serializer.is_valid():
    product = serializer.save()
    print(product.pk, serializer.data)
else:
    print(serializer.errors)
```

`data=` means incoming primitive values. `is_valid()` runs field and model-related validation; `save()` calls `create()` after successful validation. The `else` branch is essential: without it, invalid input could be mistaken for success.

# Complete Example 3

Update the price without replacing other fields:

```python
product = Product.objects.get(pk=1)
serializer = ProductSerializer(product, data={"price": "19.95"}, partial=True)
serializer.is_valid(raise_exception=True)
serializer.save()
```

The instance is the update target, and `partial=True` makes omitted required fields acceptable. Without `partial=True`, the request is treated as replacement and missing required fields fail validation. In an endpoint, retrieve the instance through a permission-scoped queryset rather than an unrestricted lookup.

# Advanced Example

For multi-row operations, wrap coupled writes in `transaction.atomic()` and enforce invariants at the database layer as well as in application validation. Add explicit permissions and pagination before exposing a production endpoint.

# Under the Hood

Routers bind HTTP verbs to action names; generic actions obtain serializers and objects through view hooks. `serializer.save()` dispatches to `create(validated_data)` or `update(instance, validated_data)`. ORM operations generate parameterized SQL; the database commits each statement unless a transaction changes the boundary.

# Performance Notes

Paginate list endpoints, select/prefetch related data used by serializers, and avoid returning unbounded querysets. Index fields used frequently for filtering or ordering after measuring real queries.

# Best Practices

- Use plural resource paths and correct status codes.
- Scope querysets before object retrieval; authentication alone is not ownership authorization.
- Keep server-controlled fields read-only and assign them from trusted request context.
- Validate at the serializer boundary and enforce critical invariants in the database.
- Document deletion semantics, pagination, and error shapes.

# Common Mistakes

- Using `PUT` as though it were a partial update.
- Returning a success response after serializer validation failed.
- Exposing every model field without reviewing data access.
- Allowing list endpoints to return all rows.

# Debugging

Check the resolved URL/action, request method/content type, `serializer.errors`, permission result, filtered queryset, and database constraint error. Reproduce requests with DRF's `APIClient` and inspect response status and body.

# Interview Questions

1. What does CRUD stand for? Create, Read, Update, Delete.
2. How do `PUT` and `PATCH` differ? Replacement semantics versus partial modification.
3. Does a ModelViewSet make an endpoint secure? No; access policy and queryset scope must be configured.
4. Why validate in both application and database layers? Useful client errors plus integrity under concurrency and alternate write paths.

# Practice Exercises

1. Add a required category and explain how create validation changes.
2. Restrict list and detail access to active products.
3. Write tests for create, retrieve, invalid input, and delete.

# Chapter Summary

CRUD is a resource lifecycle, not merely five HTTP handlers. DRF connects routing, policy, serializer validation, ORM persistence, and response serialization. Production correctness depends on access scoping, data integrity, bounded queries, and predictable semantics.

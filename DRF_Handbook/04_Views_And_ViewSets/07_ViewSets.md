# Introduction

A DRF ViewSet groups related resource actions—such as list, retrieve, create, update, and destroy—into one class. A router maps those actions to URL patterns and HTTP methods.

# Why this concept exists

Separate views often repeat resource configuration and route wiring. A ViewSet expresses the resource's operations together, while routers generate conventional routes.

# Real-world analogy

A ViewSet is a service department for one resource; the router is its directory that maps each address and method to the right desk action.

# Internal Working

```text
router registers ViewSet -> creates URL patterns/action map
HTTP method -> action (GET collection=list, GET detail=retrieve, POST=create)
             -> ViewSet dispatch -> serializer/queryset -> Response
```

# Request Lifecycle

Router-generated callable binds HTTP methods to ViewSet action names. DRF dispatch then performs request initialization, policies, action handler, exception handling, and response finalization.

# Syntax

```python
class ProductViewSet(viewsets.ViewSet):
    def list(self, request):
        ...
```

# Complete Example 1

This manually implemented ViewSet uses in-memory sample data to show action routing. It is not production persistence.

```python
# api/views.py
from rest_framework import status, viewsets
from rest_framework.response import Response

ITEMS = [{"id": 1, "name": "Pen"}]


class ItemViewSet(viewsets.ViewSet):
    def list(self, request):
        return Response({"results": ITEMS})

    def retrieve(self, request, pk=None):
        item = next((item for item in ITEMS if str(item["id"]) == str(pk)), None)
        if item is None:
            return Response({"detail": "Not found."}, status=404)
        return Response(item)

    def create(self, request):
        name = request.data.get("name")
        if not isinstance(name, str) or not name.strip():
            return Response({"name": ["Required."]}, status=400)
        item = {"id": len(ITEMS) + 1, "name": name.strip()}
        ITEMS.append(item)
        return Response(item, status=status.HTTP_201_CREATED)
```

```python
# api/urls.py
from rest_framework.routers import SimpleRouter
from .views import ItemViewSet

router = SimpleRouter()
router.register("items", ItemViewSet, basename="item")
urlpatterns = router.urls
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("api.urls"))]
```

**Line-by-line:** `ViewSet` provides action-oriented dispatch. `list`, `retrieve`, and `create` implement collection/detail operations. Input is manually checked for this compact demonstration. `SimpleRouter.register` associates resource prefix and basename, then generates list/detail routes. Project URLconf mounts them.

**Request → Router → View/ViewSet → Serializer → Database → Response flow:** router maps GET `/items/` to list, GET `/items/1/` to retrieve, POST `/items/` to create -> viewset action -> inline validation (no serializer) -> in-memory storage -> response. **Internal execution:** Router action map translates methods to names before dispatch. **Professional use case:** Resource grouping when endpoints share configuration. Replace global list and inline checks with ORM and serializers; removing router registration means no generated routes.

# Complete Example 2

Use `GenericViewSet` with mixins:

```python
class ReadCreateViewSet(
    mixins.ListModelMixin,
    mixins.CreateModelMixin,
    viewsets.GenericViewSet,
):
    queryset = Item.objects.all()
    serializer_class = ItemSerializer
```

This supplies list/create actions without all CRUD methods. Register it with a router. Use it when standard behavior fits and only a subset is needed.

# Complete Example 3

Restrict methods/actions through a limited ViewSet composition rather than exposing `ModelViewSet` by default. Review generated router routes and test 405 behavior for unsupported methods.

# Advanced Example

ViewSets expose `self.action`, useful for selecting serializer/permissions by operation. Ensure action-dependent policies cover custom actions and do not use action names as a replacement for explicit authorization.

# Under the Hood

Router calls ViewSet `as_view` with an HTTP-method-to-action mapping. The resulting view stores action metadata, then APIView dispatch proceeds with the selected handler.

# Performance Notes

ViewSet grouping does not optimize querysets. Use `get_queryset` and related-object loading; paginate list actions.

# Best Practices

- Choose `ViewSet` for fully custom actions, `GenericViewSet` for generic mixin behavior.
- Register explicit basenames where queryset is dynamic/absent.
- Keep supported actions intentional.
- Test generated route list and action policy.

# Common Mistakes

- Assuming ViewSet alone generates routes.
- Forgetting router registration/basename.
- Leaving sample global mutable state in production.
- Exposing every action unintentionally.

# Debugging

Inspect router URLs, basename, action map, `self.action`, method, permissions, and queryset. Use `show_urls` tooling if available or Django resolver tests.

# Interview Questions

1. What is a ViewSet? A class grouping actions for a resource.
2. What does router do? Generates URLs and method/action mappings.
3. ViewSet vs GenericViewSet? GenericViewSet adds generic queryset/serializer helpers.
4. How restrict actions? Compose only needed mixins/actions and inspect routes.

# Practice Exercises

1. Register a ViewSet and inspect collection/detail URLs.
2. Replace inline data checks with a serializer.
3. Remove update/delete support and test those methods return 405.

# Chapter Summary

ViewSets group resource actions and routers map them into routes. They improve organization, not security or persistence automatically; select supported actions deliberately and use serializers/querysets for real resources.

# Introduction

DRF routers generate URL patterns for ViewSets. They map collection/detail routes and HTTP methods to actions, reducing repetitive URL declarations.

# Why this concept exists

ViewSets group resource actions, and routers provide consistent route conventions for those actions across an API.

# Real-world analogy

A router is the building directory that maps each service address and action to the correct department.

# Internal Working

```text
router.register(prefix, viewset, basename)
 -> route templates -> ViewSet.as_view(method-to-action map)
 -> URL resolver -> action
```

The router does not implement the actions, authorize requests, or guarantee queryset scoping; those remain view responsibilities.

# Request Lifecycle

The router-generated URL resolves to a callable configured with an action map. DRF dispatch then runs normal policies and action logic.

# Syntax

```python
router = SimpleRouter()
router.register("books", BookViewSet, basename="book")
urlpatterns = router.urls
```

# Complete Example 1

Register a ModelViewSet with a SimpleRouter. Assume `BookViewSet` is implemented and has a queryset/serializer.

```python
# library/urls.py
from rest_framework.routers import SimpleRouter
from .views import BookViewSet

router = SimpleRouter()
router.register("books", BookViewSet, basename="book")
urlpatterns = router.urls
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("library.urls"))]
```

The router generates collection and detail routes for supported actions.

**Line-by-line:** Import SimpleRouter; instantiate it; register route prefix, viewset, and stable name prefix; expose generated URL patterns; include them under `/api/`. **Flow:** `/api/books/` + GET -> list action; `/api/books/1/` + GET -> retrieve; POST collection -> create; detail verbs -> update/delete if viewset supports them. **Internal working:** Router uses action mappings to configure ViewSet callables. **Professional tip:** Use explicit basename if the viewset's queryset is dynamic or missing. Removing URL inclusion makes routes unreachable; registering a ViewSet does not secure it.

# Complete Example 2

A viewset without class-level queryset needs an explicit basename:

```python
class MyViewSet(viewsets.GenericViewSet):
    serializer_class = BookSerializer

    def get_queryset(self):
        return Book.objects.filter(owner=self.request.user)

router.register("my-books", MyViewSet, basename="my-book")
```

Router cannot infer a model basename from a dynamic method-only queryset. Explicit naming supports reverse route names such as `my-book-list`.

# Complete Example 3

Include extra router URLs from another app:

```python
urlpatterns = [
    path("api/library/", include("library.urls")),
    path("api/orders/", include("orders.urls")),
]
```

Each router owns its resource routes. Namespaces can help avoid reverse-name collisions across apps.

# Advanced Example

Inspect generated patterns with Django URL resolver tools or `show_urls` in development. Router route ordering and lookup regex/converters can be customized; avoid overlapping patterns that make routes ambiguous.

# Under the Hood

Router registry stores prefix, viewset, basename, and optional detail behavior. URL generation creates route patterns and method/action dictionaries passed to `ViewSet.as_view`.

# Performance Notes

Router generation happens during configuration; request performance is governed by view/query/serializer behavior, not by route registration.

# Best Practices

- Use clear resource prefixes and basenames.
- Register only intended ViewSets/actions.
- Scope data in viewsets independently of URL shape.
- Test named route reversal and generated method behavior.

# Common Mistakes

- Forgetting to include `router.urls`.
- Missing basename for dynamic queryset.
- Assuming route registration grants permission.
- Using inconsistent pluralization and route naming.

# Debugging

Inspect router registry, `router.urls`, URL resolver, basename, and action map. Verify exact mounted prefix and trailing slash policy.

# Interview Questions

1. What does a router generate? URL patterns and HTTP method/action maps.
2. What is basename? Prefix for generated route names.
3. Why explicit basename? Router may not infer a model from dynamic queryset.
4. Does router handle permissions? No.

# Practice Exercises

1. Register a BookViewSet and reverse list/detail names.
2. Build a dynamic queryset viewset with explicit basename.
3. Test POST route exists only when create action is supported.

# Chapter Summary

Routers connect ViewSet actions to conventional URLs. They automate routing only; permissions, query scoping, serializers, and business rules remain explicit view responsibilities.

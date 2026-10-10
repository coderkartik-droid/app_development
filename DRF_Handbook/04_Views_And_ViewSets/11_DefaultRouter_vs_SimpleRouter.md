# Introduction

`SimpleRouter` generates standard ViewSet collection/detail routes. `DefaultRouter` provides those routes and adds an API root view that links registered resources; it may also support format suffix patterns depending on configuration/version.

# Why this concept exists

Projects differ in whether they need a navigable root index. Choosing the smaller router can keep routing surface minimal.

# Real-world analogy

SimpleRouter is a directory with direct office addresses. DefaultRouter adds a lobby directory pointing to each office.

# Internal Working

Both routers inspect registered ViewSets and generate URL patterns from available actions. DefaultRouter adds a root endpoint that enumerates registered route links.

# Request Lifecycle

For resource routes, both routers map HTTP methods to actions similarly. DefaultRouter's root route is an additional view; it does not alter resource permissions or data behavior.

# Syntax

```python
router = SimpleRouter()
# or
router = DefaultRouter()
```

# Complete Example 1

Choose SimpleRouter for a project that does not need a generated API root:

```python
# api/urls.py
from rest_framework.routers import SimpleRouter
from .views import ProductViewSet

router = SimpleRouter()
router.register("products", ProductViewSet, basename="product")
urlpatterns = router.urls
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("api.urls"))]
```

**Line-by-line:** Instantiate SimpleRouter, register a resource, expose its patterns, and mount them at `/api/`. It generates collection/detail paths but no root listing. **Flow:** resource path -> generated pattern -> action map -> viewset -> serializer/query -> response. **Internal working:** The router derives routes from ViewSet actions. **Professional tip:** Pick SimpleRouter for a minimal API surface; clients should use documented URLs rather than depend on a browsable index. Removing registration eliminates product routes.

# Complete Example 2

Choose DefaultRouter when a discoverable root index is useful:

```python
from rest_framework.routers import DefaultRouter

router = DefaultRouter()
router.register("products", ProductViewSet, basename="product")
urlpatterns = router.urls
```

The root may be available at the mounted `/api/` and link to product routes. It helps humans explore in development; it is not a substitute for API documentation.

# Complete Example 3

Test resource behavior identically regardless of router:

```python
response = self.client.get(reverse("product-list"))
self.assertEqual(response.status_code, 200)
```

Route names are based on the registered basename. Prefer `reverse` so router changes do not require test path edits.

# Advanced Example

DefaultRouter may generate format-suffix variants based on DRF configuration/version. If the project disables suffixes or uses custom patterns, expectations differ; inspect generated URL patterns rather than assuming.

# Under the Hood

DefaultRouter extends SimpleRouter and adds an API root view plus configured suffix behavior. Resource action mappings remain based on the registered ViewSet.

# Performance Notes

Router selection has negligible request cost for ordinary routes. The API root can add one endpoint but does not affect data endpoint query efficiency.

# Best Practices

- Use SimpleRouter for minimal routing.
- Use DefaultRouter when an API root improves discoverability.
- Document stable endpoints independently of root representation.
- Test generated URL names and trailing-slash policy.

# Common Mistakes

- Expecting SimpleRouter to expose an API root.
- Treating DefaultRouter root as complete documentation.
- Assuming resource permissions are inherited from root visibility.
- Hard-coding router-dependent URL strings everywhere.

# Debugging

Inspect `router.urls`, reverse names, mounted prefix, and router options. Confirm installed DRF version for suffix/root behavior.

# Interview Questions

1. Main difference? DefaultRouter adds API root and may add suffix patterns.
2. Does SimpleRouter generate CRUD resource routes? It generates routes for supported ViewSet actions.
3. Is API root required? No.
4. Do routers enforce permissions? No.

# Practice Exercises

1. Implement each router and compare generated URLs.
2. Test reverse lookup for list/detail.
3. Decide which router suits an internal API and justify.

# Chapter Summary

Use SimpleRouter for standard routes only, DefaultRouter when a root index helps exploration. Their core resource routing is similar; neither defines authorization or API documentation policy.

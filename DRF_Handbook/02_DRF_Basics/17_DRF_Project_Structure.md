# Introduction

A maintainable DRF project separates project-wide configuration from domain apps, and organizes API code so routes, serializers, views, models, and tests are easy to locate.

# Why this concept exists

As endpoints grow, a single large `views.py` becomes hard to navigate and review. Clear structure communicates ownership and reduces accidental coupling.

# Real-world analogy

A project structure is an office directory: shared policies are centralized, while each department keeps its own files and workflows.

# Internal Working

```text
config/settings.py -> app registry + global DRF config
config/urls.py -> app URLconf -> view -> serializer/service -> model/ORM
```

Django discovers apps through `INSTALLED_APPS`; URL includes compose endpoint routes; imports connect API components at runtime.

# Request Lifecycle (where applicable)

Folder layout does not alter request processing. A request still passes middleware, root/app URL resolution, view dispatch, optional serializer and ORM, then response rendering.

# Syntax

```text
project/
  manage.py
  config/
    settings.py
    urls.py
  catalog/
    models.py
    serializers.py
    views.py
    urls.py
    tests.py
```

# Complete Example 1

For a small domain app, use Django's conventional files and add DRF components:

```text
catalog/
  apps.py
  models.py
  serializers.py
  views.py
  urls.py
  tests.py
```

```python
# config/urls.py
from django.urls import include, path

urlpatterns = [path("api/catalog/", include("catalog.urls"))]
```

```python
# catalog/urls.py
from django.urls import path
from .views import health

urlpatterns = [path("health/", health, name="health")]
```

**Line-by-line:** The tree groups code by app/domain. Root URLconf imports `include` and `path`, then mounts the app route list beneath a prefix. The app URLconf owns its local endpoint mapping. **Flow:** `/api/catalog/health/` -> root prefix -> app route -> view. **Internal working:** Django imports URL modules during resolver setup and dispatches only after a match. **Professional tip:** Begin with conventional modules; split into packages when file size or domain complexity justifies it. A folder tree alone does not register an app or route.

# Complete Example 2

For a larger app, group by feature:

```text
catalog/
  api/
    serializers.py
    views.py
    urls.py
  domain/
    services.py
  models.py
  tests/
    test_api.py
```

The API package contains transport concerns while domain services hold reusable business operations. **Flow:** URL -> API view -> service -> ORM -> serializer/response. **Tip:** Avoid creating layers that merely forward calls without adding clarity, validation, or reuse.

# Complete Example 3

Register an app config in settings:

```python
INSTALLED_APPS = [
    "rest_framework",
    "catalog.apps.CatalogConfig",
]
```

This makes DRF and the app available to Django's app registry. **Tip:** Verify the active settings module in tests and deployment; the same source can behave differently under different settings.

# Advanced Example

For multi-app projects, use URL namespaces and clear ownership for shared authentication, error handling, and settings. Separate API versioning only when a compatibility strategy exists; directories named `v1` and `v2` without policy are not versioning by themselves.

# Under the Hood

Python import paths define module identity. Django's app registry imports configured app modules; URL resolvers load URL patterns. Circular imports can arise if modules depend on each other at import time.

# Performance Notes

Project organization itself does not improve endpoint speed. Keep ORM query construction and serialization efficient, and avoid expensive top-level work during module import.

# Best Practices

- Keep project config separate from app features.
- Group endpoints around domain ownership.
- Keep URLs and tests close to the feature they cover.
- Avoid import-time database queries and side effects.

# Common Mistakes

- One enormous app for unrelated domains.
- Excessive layers and packages before needed.
- Forgetting URL inclusion or app registration.
- Treating directories as a substitute for clear responsibilities.

# Debugging

Check `INSTALLED_APPS`, import paths, root URL includes, app URL patterns, and active settings. Use Django's URL resolver/reverse and system checks to find configuration errors.

# Interview Questions

1. Where place project-wide settings? Project configuration package.
2. Where put domain API routes? In the owning app's URLconf.
3. Does structure alter DRF lifecycle? No.
4. What causes circular imports? Modules importing each other during initialization without a clear dependency direction.

# Practice Exercises

1. Create a `catalog` app with separate model, serializer, view, URL, and test modules.
2. Mount routes under `/api/catalog/`.
3. Add a namespace and reverse a named route.

# Chapter Summary

Keep project configuration centralized and domain code cohesive. Conventional app modules are a good start; introduce deeper structure only when it makes responsibilities clearer.

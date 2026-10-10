# Introduction

A Django app is a Python package for a focused feature. `startapp` creates conventional modules for models, views, tests, admin, and app configuration.

# Why this concept exists

Apps let a project organize domain capabilities and make components reusable. DRF APIs are commonly grouped inside Django apps.

# Real-world analogy

An app is a department with its own records, workflows, and staff, connected to the wider organization through defined interfaces.

# Internal Working

`startapp` creates files; adding the app to `INSTALLED_APPS` lets Django discover its configuration and models. App URLs are included from the project URLconf when needed.

```text
create app -> register app -> define models/views/routes -> Django loads app
```

# Syntax

```bash
python manage.py startapp catalog
```

# Complete Example 1

Assuming a project exists and its virtual environment is active:

```bash
python manage.py startapp catalog
```

```python
# project/settings.py
INSTALLED_APPS = [
    # ...
    "catalog.apps.CatalogConfig",
]
```

**Line-by-line:** `startapp catalog` creates the app package. The dotted `CatalogConfig` path registers the app's metadata class. Django loads it at startup and discovers app models/migrations. **Execution flow:** management command creates files -> settings register app -> framework imports app config -> app participates in setup. **Professional tip:** Keep app names stable after migrations and deployments.

# Complete Example 2

Add one route in the app:

```python
# catalog/views.py
from django.http import HttpResponse


def index(request):
    return HttpResponse("Catalog")
```

```python
# catalog/urls.py
from django.urls import path
from .views import index

urlpatterns = [path("", index, name="index")]
```

```python
# project/urls.py
from django.urls import include, path

urlpatterns = [path("catalog/", include("catalog.urls"))]
```

The app view returns a response; its URLconf maps an empty suffix; the project mounts it at `/catalog/`. **Internal:** `include` delegates matching to the app route list. **Tip:** Namespace URL names as project grows.

# Advanced Example

An app may define models, serializers, API views, URLs, admin registrations, and tests while remaining cohesive around one domain. Avoid circular imports by keeping dependencies flowing in a clear direction.

# Common Mistakes

- Forgetting app registration.
- Creating app URLs but never including them.
- Naming apps after generic words that conflict with packages.
- Combining unrelated domain capabilities into an unmaintainable app.

# Best Practices

- Use clear, singular/plural naming consistently.
- Add app-specific tests and URLs.
- Keep app boundaries focused and explicit.
- Avoid startup-time database work in `AppConfig.ready`.

# Interview Questions

1. What does `startapp` create? A conventional app package structure.
2. How does Django load it? Through `INSTALLED_APPS`.
3. How mount app routes? Include the app URLconf in project URLs.
4. What makes a good app boundary? A cohesive domain responsibility.

# Practice

Create a `books` app, register its config class, and mount its URLconf at `/books/`.

# Summary

Create apps with `startapp`, register them, and connect their routes intentionally. Cohesive apps make a DRF project easier to navigate and test.

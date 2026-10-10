# Introduction

This summary collects the Django concepts needed before beginning Django REST Framework: project/app organization, routing, views, models, ORM, migrations, settings, and input validation.

# Why this concept exists

DRF builds on Django rather than replacing it. Understanding Django's request and data layers makes DRF serializers and API views easier to learn.

# Real-world analogy

Django is the foundation and structure of a building; DRF adds an API service counter with specialized request parsing and response handling.

# Internal Working

```text
project settings -> middleware -> URLconf -> view
                                  -> ORM/model (optional)
                                  -> template/response
```

DRF later enhances the view/request/representation layer while continuing to use Django settings, routing, models, and migrations.

# Syntax

```bash
python manage.py startapp catalog
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```

# Complete Example 1

A compact plain Django route:

```python
# catalog/views.py
from django.http import JsonResponse


def health(request):
    return JsonResponse({"status": "ok"})
```

```python
# catalog/urls.py
from django.urls import path
from .views import health

urlpatterns = [path("health/", health, name="health")]
```

```python
# project/urls.py
from django.urls import include, path

urlpatterns = [path("api/", include("catalog.urls"))]
```

**Line-by-line:** The view constructs a JSON response; app URLconf names the route; project URLconf mounts it under `/api/`. **Flow:** request -> root URLconf -> app URLconf -> view -> JSON response. **Internal:** This is Django routing/response, not yet DRF serialization. **Professional tip:** Use this mental model to trace any endpoint before debugging higher-level behavior.

# Complete Example 2

The model/ORM lifecycle:

```python
class Product(models.Model):
    name = models.CharField(max_length=120)
```

```bash
python manage.py makemigrations
python manage.py migrate
```

`Product` defines a field; migration generation records the schema operation; migration application updates the configured database. **Flow:** model -> migration -> SQL schema -> ORM queryset. **Tip:** Never assume a model edit is deployed until its migration is applied.

# Advanced Example

For API work, keep responsibilities distinguishable: Django model/database rules; serializer input validation and representation; view request orchestration; URL routing; permission policy. DRF uses Django's app registry/settings and ORM, so these foundations remain relevant.

# Common Mistakes

- Treating project and app as synonyms.
- Forgetting `INSTALLED_APPS` or URL inclusion.
- Confusing templates with API serialization.
- Skipping migrations or input validation.

# Best Practices

- Trace request -> URL -> view -> data -> response.
- Keep apps cohesive and views focused.
- Use migrations and database constraints.
- Learn the distinction between Django forms and DRF serializers.

# Interview Questions

1. What routes a request? URLconf.
2. What represents database structure? Models and migrations.
3. What handles a request? A view.
4. What is DRF's role? Build structured APIs on top of Django.

# Practice

Without looking back, draw the request path and explain project, app, view, model, ORM, and migration roles.

# Summary

Django provides project configuration, app organization, request routing, views, ORM models, and migrations. DRF adds API-focused representation, validation, and request handling on top.

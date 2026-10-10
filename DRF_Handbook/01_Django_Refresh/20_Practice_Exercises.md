# Introduction

These short exercises check whether Django foundations are clear enough to begin DRF. Focus on request flow, app setup, model persistence, and safe input handling.

# Why this concept exists

DRF introduces new abstractions. Practicing Django basics first prevents confusion about which behavior comes from Django and which comes from DRF.

# Real-world analogy

Before learning to drive a delivery van, practice steering, braking, and reading road signs in a simple car.

# Internal Working

Each exercise follows the same path: configure the app, route a request, run view logic, access data if needed, and return a response.

```text
settings -> URLconf -> view -> form/model/ORM -> response -> test
```

# Syntax

```python
urlpatterns = [path("books/", views.book_list, name="book-list")]
```

# Complete Example 1

Build a status endpoint:

```python
# practice/views.py
from django.http import JsonResponse


def status(request):
    return JsonResponse({"status": "learning"})
```

```python
# practice/urls.py
from django.urls import path
from .views import status

urlpatterns = [path("status/", status, name="status")]
```

Add `"practice"` to `INSTALLED_APPS` and include `practice.urls` from the project URLconf. **Line-by-line:** The view returns a JSON mapping; the URL pattern names the endpoint; app registration makes Django load it; root inclusion mounts the route. **Flow:** request -> root include -> local route -> view -> JSON. **Internal:** URL routing and response generation are Django fundamentals used by DRF too. **Professional tip:** Test response status and data shape, not only that the route loads.

# Complete Example 2

Build a tiny model/ORM exercise:

```python
# practice/models.py
from django.db import models


class Bookmark(models.Model):
    title = models.CharField(max_length=100)
```

```bash
python manage.py makemigrations practice
python manage.py migrate
python manage.py shell
```

In the shell: `Bookmark.objects.create(title="Django docs")`, then `Bookmark.objects.filter(title__icontains="django").count()`. **Line-by-line:** The model declares a bounded title; migration commands create/apply the table; ORM creates a row and counts matches. **Flow:** model -> migration -> database -> queryset -> result. **Tip:** Add tests and a uniqueness rule only if the domain requires it.

# Advanced Example

Exercise: replace a hand-written plain JSON view with a DRF serializer and API view in the next folder. Compare validation, request parsing, status handling, and response objects; do not duplicate Django's entire infrastructure.

# Common Mistakes

- Forgetting app registration or URL inclusion.
- Running ORM commands before migrations.
- Treating request input as trusted.
- Confusing a successful route match with a correct response.

# Best Practices

- Use an isolated environment and reproducible dependencies.
- Run `manage.py check` and tests.
- Keep data constraints in models/database where appropriate.
- Use named routes and clear app boundaries.

# Interview Questions

1. What does a migration do? Applies versioned schema changes.
2. What is a QuerySet? A composable, typically lazy ORM query.
3. What does an app provide? A focused reusable feature package.
4. What is the first request-routing step? URL pattern resolution.

# Practice

1. Create and register a `library` app.
2. Add a named route returning JSON.
3. Add a `Book` model and migrate it.
4. Create and query one row through the ORM.
5. Add a form or validation rule and test invalid input.

# Summary

Be ready to create a project/app, register routes, write a view, define a model, apply migrations, and query through the ORM. These are the Django foundations DRF builds upon.

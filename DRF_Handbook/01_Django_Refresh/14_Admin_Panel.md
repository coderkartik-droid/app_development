# Introduction

Django admin is a built-in staff interface for managing registered models. It is valuable for development and trusted internal workflows, but is not a replacement for a purpose-designed public interface.

# Why this concept exists

Developers and authorized staff often need a quick way to inspect and edit database records without building a custom interface first.

# Real-world analogy

Admin is a staff-only control room: useful for authorized operators, but not the customer-facing storefront.

# Internal Working

Django discovers registered models and creates model forms/list pages. Authentication and model permissions gate access; `ModelAdmin` controls display, search, filters, and permitted actions.

```text
staff login -> admin URL -> permission check -> ModelAdmin -> ORM operations
```

# Syntax

```python
@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    list_display = ("title", "published_year")
```

# Complete Example 1

Register a model and configure useful list columns:

```python
# catalog/admin.py
from django.contrib import admin
from .models import Book


@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    list_display = ("title", "published_year")
    search_fields = ("title",)
    list_filter = ("published_year",)
```

Create an admin user with `python manage.py createsuperuser`, apply migrations, then run the development server and visit `/admin/`. **Line-by-line:** Import admin and model; decorator registers `Book`; `list_display` chooses columns; `search_fields` enables title search; `list_filter` adds a year filter. **Flow:** staff authenticates -> admin checks permissions -> query is built -> records render/edit. **Internal:** Admin uses the ORM and model metadata. **Professional tip:** Restrict staff accounts and permissions; never expose admin as a public API.

# Complete Example 2

Limit editable fields:

```python
@admin.register(Book)
class BookAdmin(admin.ModelAdmin):
    fields = ("title", "published_year")
    readonly_fields = ("published_year",)
```

The form displays the chosen fields and makes the year read-only in this admin form. This is an admin UI constraint, not a universal database invariant; enforce critical rules in models/database as appropriate. **Tip:** Read-only presentation does not replace authorization or validation.

# Advanced Example

Override `get_queryset` or permissions only when the policy is well understood and tested. Admin queryset filtering can improve usability but must not be treated as the only security boundary for API views.

# Common Mistakes

- Forgetting to register a model.
- Creating a superuser before migrations are applied.
- Giving every staff user unrestricted model permissions.
- Treating admin form restrictions as global data protection.

# Best Practices

- Configure search and filters for staff workflows.
- Grant least privilege.
- Use admin actions carefully for bulk mutations.
- Keep admin-specific behavior separate from public API permissions.

# Interview Questions

1. What is Django admin? A model-driven staff management interface.
2. How register a model? `admin.site.register` or `@admin.register`.
3. Does admin replace a product UI? No.
4. Are admin restrictions enough to secure an API? No; API permission checks are separate.

# Practice

Register a model with list display, search, and filtering. Create a staff user and inspect permission behavior.

# Summary

Django admin accelerates trusted internal data management. Register models, tailor workflows, and apply least privilege; build separate public API behavior.

# Introduction

A Django model is a Python class that describes stored data. Django's ORM maps model fields to database columns and provides a Python interface for creating and querying records.

# Why this concept exists

Apps need durable structured records with constraints and relationships. Models centralize the schema description and connect Python code to database operations.

# Real-world analogy

A model is a blueprint for a filing card: it defines the fields every card has and the rules those fields must follow.

# Internal Working

At startup, Django discovers models in installed apps. Migrations translate model schema changes into database operations. The ORM builds SQL from queryset operations and maps result rows back to model instances.

```text
Python model -> migrations -> SQL schema
QuerySet -> SQL -> database rows -> model instances
```

# Syntax

```python
class Book(models.Model):
    title = models.CharField(max_length=200)
    published_year = models.PositiveSmallIntegerField(null=True, blank=True)
```

# Complete Example 1

Create the model in an installed app, then generate and apply migrations.

```python
# catalog/models.py
from django.db import models


class Book(models.Model):
    title = models.CharField(max_length=200)
    published_year = models.PositiveSmallIntegerField(null=True, blank=True)

    def __str__(self):
        return self.title
```

```bash
python manage.py makemigrations catalog
python manage.py migrate
```

**Line-by-line:** Import Django's model API. Define `Book` as a database-backed model. `CharField` stores bounded text; `PositiveSmallIntegerField` allows an optional year; `null` controls database nullability and `blank` controls validation. `__str__` gives a readable label. `makemigrations` creates a schema-change file; `migrate` applies it. **Flow:** model declaration -> migration operation -> SQL table -> ORM records. **Professional tip:** Add validators/constraints for domain rules and don't equate form blankness with database nullability.

# Complete Example 2

Create and query a record:

```python
book = Book.objects.create(title="Django Essentials", published_year=2025)
matches = Book.objects.filter(title__icontains="django").order_by("title")
```

`create` inserts and returns a model instance. `filter` constructs a lazy queryset; evaluation occurs when consumed. `order_by` sets database sorting. **Internal:** The ORM compiles these operations to parameterized SQL and converts rows to `Book` instances. **Tip:** Use `select_related`/`prefetch_related` for relationship access patterns.

# Advanced Example

Define a uniqueness constraint with `Meta.constraints` when uniqueness is a data invariant, then create/apply a migration. Database constraints protect integrity across admin, scripts, and API code—not just one form.

# Common Mistakes

- Editing models without making and applying migrations.
- Using `null=True` on text fields without a clear need.
- Returning all rows for unbounded datasets.
- Assuming model validation alone enforces every database invariant.

# Best Practices

- Choose field types and nullability intentionally.
- Add constraints and indexes based on domain/query needs.
- Use migrations for every released schema change.
- Keep `__str__` useful for admin and debugging.

# Interview Questions

1. What is a Django model? A class mapping domain data to database structure.
2. What is a QuerySet? A lazy, composable database query representation.
3. Why migrations? To version and apply schema changes.
4. Difference between `null` and `blank`? Database nullability vs validation/form emptiness.

# Practice

Create a `Category` model with a unique name and relate books to categories. Generate a migration.

# Summary

Models describe stored records; migrations apply schema changes; the ORM translates Python queries into SQL. Constraints belong at the database boundary when they represent invariants.

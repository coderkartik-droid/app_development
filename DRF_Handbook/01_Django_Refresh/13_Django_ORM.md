# Introduction

Django's Object-Relational Mapper (ORM) lets Python code define models and query database records without writing SQL for every operation. It still uses SQL underneath.

# Why this concept exists

The ORM provides a consistent, composable interface for common data operations, maps rows to Python objects, and integrates with migrations and relationships.

# Real-world analogy

The ORM is a translator between Python requests and the database's SQL language. You ask for books matching a condition; it builds the database query and translates rows back.

# Internal Working

```text
QuerySet expression -> ORM compiler -> parameterized SQL -> database -> model objects
```

QuerySets are usually lazy: composing `.filter()` does not necessarily execute SQL until evaluation, iteration, or conversion.

# Syntax

```python
Book.objects.filter(published_year__gte=2020).order_by("-published_year")
```

# Complete Example 1

With `Book` defined and migrations applied:

```python
# catalog/views.py
from django.http import JsonResponse
from .models import Book


def recent_books(request):
    books = Book.objects.filter(
        published_year__gte=2020,
    ).order_by("-published_year", "title")
    data = [
        {"id": book.id, "title": book.title, "year": book.published_year}
        for book in books
    ]
    return JsonResponse({"results": data})
```

Connect `recent_books` to a URL. **Line-by-line:** Import the response and model; construct a queryset filtered by year; order newest year first and title as tie-breaker; iterate to evaluate the query and serialize selected fields; return JSON. **Flow:** view -> queryset -> SQL `WHERE`/`ORDER BY` -> rows -> Python values -> response. **Internal:** Django binds values separately from SQL text. **Professional tip:** Add a result limit or pagination for potentially large tables; DRF pagination is covered later.

# Complete Example 2

Create and update through the ORM:

```python
book = Book.objects.create(title="API Design", published_year=2024)
book.title = "API Design Basics"
book.save(update_fields=["title"])
```

`create` inserts; assigning changes the in-memory instance; `save(update_fields=...)` persists only the named field. **Flow:** Python instance -> ORM write -> SQL insert/update -> database. **Tip:** For concurrent inventory/count updates, use database expressions and transactions rather than read-modify-write in Python.

# Advanced Example

For a foreign-key relation, `select_related("author")` can load related single-valued rows in one join; `prefetch_related("tags")` loads multi-valued relations efficiently. Use query-count tests or debug tooling to detect N+1 patterns.

# Common Mistakes

- Assuming a lazy queryset has already executed.
- Iterating related objects without considering N+1 queries.
- Loading unbounded tables into memory.
- Assuming changing a model instance automatically saves it.

# Best Practices

- Filter and order in the database.
- Select only needed related data with appropriate loading strategies.
- Use transactions for multi-step invariants.
- Inspect generated SQL when query behavior is unclear.

# Interview Questions

1. What is a QuerySet? A lazy, composable representation of a database query.
2. Does ORM mean no SQL? No; ORM generates SQL.
3. How avoid N+1 queries? Appropriate `select_related`/`prefetch_related`.
4. What does `update_fields` do? Restricts fields written by `save`.

# Practice

Create a queryset for books from 2020 onward ordered by year and title. Explain when it is evaluated.

# Summary

The ORM translates composable Python query expressions into SQL and maps results into models. Understand query laziness, result bounds, and related-object loading.

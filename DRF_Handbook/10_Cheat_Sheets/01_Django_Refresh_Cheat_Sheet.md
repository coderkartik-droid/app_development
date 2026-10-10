# Django Refresh Cheat Sheet

## Key Concepts

- **Project:** settings, root URLs, ASGI/WSGI entry points.
- **App:** reusable domain area with models, views, URLs, tests.
- **MTV:** Model (data), Template (presentation), View (request orchestration).
- **ORM:** Python QuerySets translated to parameterized SQL.
- **Migration:** versioned schema change.

## Important Syntax

```python
class Book(models.Model):
    title = models.CharField(max_length=160)

Book.objects.filter(title__icontains="django").order_by("title")
```

```shell
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```

## Frequently Used Classes and Methods

| API | Purpose |
|---|---|
| `models.Model` | Persistent entity |
| `path()` / `include()` | URL routing |
| `HttpRequest` / `HttpResponse` | Django request/response |
| `get_object_or_404()` | Safe missing-object lookup |
| `QuerySet.filter()` | Database filtering |
| `transaction.atomic()` | Database transaction |

## Common Decorators

- `@login_required`: website view requires login.
- `@require_http_methods([...])`: restrict Django view methods.
- DRF decorators such as `@api_view` belong to API views.

## Request/Response Flow

`client -> middleware -> URL resolver -> view -> ORM/template -> response -> middleware`.

## Serializer Flow

Django itself does not supply DRF serializers: API input/output is handled by DRF serializer classes in the API layer.

## View/ViewSet Comparison

| Django view | DRF view |
|---|---|
| Returns `HttpResponse`/template | Returns negotiated `Response` |
| Django request | DRF `Request` features |
| Website-oriented | API-oriented parsing, auth, permissions |

## HTTP Status Codes

`200` success · `201` created · `204` no body · `400` invalid input · `401` unauthenticated · `403` forbidden · `404` missing · `500` unexpected server failure.

## Authentication Flow

Django auth backends verify credentials; middleware associates session user with a request. DRF authenticators adapt credentials for API requests.

## JWT Flow

JWT is not a core Django feature: login issues signed access/refresh tokens; clients send access token; refresh obtains replacement access.

## Common Mistakes

- Editing models but forgetting migrations.
- Hard-coding `User` instead of using `settings.AUTH_USER_MODEL`.
- Running development server in production.
- Using raw SQL string interpolation.

## Best Practices

- Define custom user model before initial migrations if needed.
- Keep settings environment-specific and secrets external.
- Add DB constraints and indexes based on domain/query patterns.

## Interview Revision

1. Project vs app? Configuration umbrella vs modular domain component.
2. What is a QuerySet? Lazy, composable database query representation.
3. What does migration do? Evolves schema in a versioned sequence.

## Memory Tricks

- **MTV:** Model stores, View decides, Template displays.
- **Migrate:** make migration file, then migrate database.

## One-Page Summary

Django routes requests through middleware and URL patterns to views. Models define persistence; ORM QuerySets build SQL; migrations evolve schema. DRF builds API-specific parsing, serialization, authentication, permissions, and responses on top of Django.

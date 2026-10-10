# Views and ViewSets Cheat Sheet

## Key Concepts

- Views orchestrate request, policy, query/serializer, and response.
- Generic views add queryset/serializer/object helpers.
- Mixins provide reusable CRUD actions.
- ViewSets group resource actions; routers map URLs and methods.

## Important Syntax

```python
class ProductViewSet(ModelViewSet):
    serializer_class = ProductSerializer
    permission_classes = [IsAuthenticated]
    queryset = Product.objects.all()
```

## Frequently Used Classes and Methods

| Class/hook | Purpose |
|---|---|
| `APIView` | Explicit `get`/`post` handlers |
| `GenericAPIView` | Queryset and serializer helpers |
| `ListAPIView` / `RetrieveAPIView` | Read-only generic operations |
| `ModelViewSet` | Standard CRUD action set |
| `get_queryset()` | Request-scoped data |
| `get_serializer_class()` | Dynamic representation |
| `perform_create()` | Trusted create-time values |
| `get_object()` | Scoped lookup + object checks |

## Common Decorators

- `@action(detail=True/False, methods=[...])`: extra ViewSet operation.
- `@api_view([...])`: function-based API view.
- `@classmethod`/`@staticmethod`: Python methods, not DRF route decorators.

## HTTP Status Codes

GET 200 · POST create 201 · PUT/PATCH usually 200 · DELETE 204 · invalid 400 · denied 403/401 · missing 404.

## Request/Response Flow

`URL/router -> action/method -> auth -> permissions -> parser -> handler -> exception/Response -> renderer`.

## Serializer Flow

Create passes `request.data`; update passes `instance` and data; PATCH sets `partial=True`; output comes from `serializer.data`.

## View/ViewSet Comparison

| FBV/APIView | Generic view | ViewSet |
|---|---|---|
| Explicit handling | CRUD building blocks | Resource action grouping |
| Fine-grained control | Reusable common flow | Router can generate routes |

## Authentication Flow

Authentication classes set identity; permission classes authorize; `get_queryset()` scopes list/detail objects.

## JWT Flow

Configure JWTAuthentication, send bearer access, and use refresh route separately.

## Common Mistakes

- Assuming router adds permissions.
- Unscoped `queryset = Model.objects.all()` for private data.
- Forgetting custom object lookup permission checks.
- Overriding full handlers when a `perform_*` hook suffices.

## Best Practices

- Choose least complex fitting abstraction.
- Keep queryset scope request-aware.
- Use `@action` for named domain operations.
- Test generated routes and every action.

## Interview Revision

Router maps HTTP methods to ViewSet actions; it does not implement authorization.

## Memory Tricks

- **APIView: HTTP verbs. ViewSet: resource actions.**
- `perform_*` hooks customize saves; `get_*` hooks customize selection.

## One-Page Summary

Use APIView for explicit endpoint logic, generic views/mixins for reusable CRUD, and ViewSets for grouped resources. Secure querysets and permissions explicitly; routers only wire routes.

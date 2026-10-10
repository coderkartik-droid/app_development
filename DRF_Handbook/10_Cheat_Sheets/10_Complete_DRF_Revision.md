# Complete DRF Revision

## Key Concepts

| Layer | Responsibility |
|---|---|
| Django URL | Route request |
| DRF view | Orchestrate API operation |
| Authentication | Establish identity |
| Permission | Authorize action/object |
| Serializer | Validate and represent data |
| ORM/database | Query and persist durable state |
| Renderer | Encode response |

## Important Syntax

```python
class ItemViewSet(ModelViewSet):
    serializer_class = ItemSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Item.objects.filter(owner=self.request.user)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```

## Frequently Used Classes and Methods

`APIView`, `GenericAPIView`, `ModelViewSet`, `ModelSerializer`, `Response`, `BasePermission`, `APIClient`, `transaction.atomic`, `select_related`, `prefetch_related`.

## Common Decorators

`@api_view`, `@action`, `@permission_classes`, `@authentication_classes`, `@receiver`, `@shared_task`.

## HTTP Status Codes

| Code | Meaning |
|---|---|
| 200 | Success with response |
| 201 | Created |
| 202 | Accepted for later work |
| 204 | Success, no body |
| 400 | Invalid input |
| 401 | Credentials required/invalid |
| 403 | Policy denied |
| 404 | Missing or intentionally hidden |
| 409 | State/resource conflict |
| 429 | Too many requests |
| 500 | Unexpected server failure |

## Request/Response Flow

```text
Client -> URL/router -> DRF request -> auth -> permission/throttle
-> view/action -> serializer/queryset -> domain/ORM/database
<- response/status <- renderer <- logs/metrics
```

## Serializer Flow

`data=...` -> field coercion -> field validation -> object validation -> `validated_data` -> `save()` -> create/update. Output is `serializer(instance).data`.

## View/ViewSet Comparison

APIView is explicit; generic views compose common behavior; ViewSets group resource actions; routers generate mappings but do not authorize.

## Authentication Flow

Session uses server session + CSRF; Basic sends reusable password and needs TLS; DRF token uses token record; JWT validates signed bearer claims. Authentication identifies; permission authorizes.

## JWT Flow

Login issues access + refresh -> access to resource API -> refresh only to renewal endpoint -> rotate/revoke per policy. JWT claims are readable unless separately encrypted; signature ensures integrity, not secrecy.

## Common Mistakes

- Unscoped querysets and client-trusted ownership.
- Unbounded list responses.
- Blind retries of non-idempotent POST.
- Broad exception swallowing.
- Misconfigured cache privacy or upload validation.

## Best Practices

- Deny by default; scope querysets.
- Explicit serializer fields and DB constraints.
- Paginate and measure queries.
- Make cross-system work idempotent/outbox-backed.
- Use secure secrets, HTTPS, safe logs, tests, and deployment checks.

## Interview Revision

- PUT replaces; PATCH changes supplied fields.
- `select_related` uses joins; `prefetch_related` batches related queries.
- Object checks do not filter list rows.
- `on_commit` avoids enqueue on rollback but does not alone guarantee publication.

## Memory Tricks

- **N-A-P-V:** Negotiate, Authenticate, Permission, View.
- **I-V-S:** Instantiate serializer, Validate, Save.
- **Identity is not access; signed is not secret; atomic DB is not distributed atomicity.**

## One-Page Summary

DRF maps HTTP requests into policy-checked view operations, uses serializers for validation/representation, and persists through Django ORM/database. Production APIs add scoped authorization, stable contracts, bounded queries, secure credential lifecycles, resilient async work, observability, comprehensive tests, and safe deployment practices.

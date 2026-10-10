# DRF Interview Cheat Sheet

## Key Concepts

- DRF extends Django with API requests/responses, serializers, auth, permissions, and generic views.
- QuerySets are lazy.
- Database constraints protect concurrent invariants.
- Authentication != authorization.

## Important Syntax

```python
serializer = ItemSerializer(data=request.data)
serializer.is_valid(raise_exception=True)
item = serializer.save(owner=request.user)
```

## Frequently Used Classes and Methods

`APIView`, `GenericAPIView`, `ModelViewSet`, `BasePermission`, `ModelSerializer`, `get_queryset`, `get_object`, `perform_create`, `transaction.atomic`.

## Common Decorators

`@api_view`, `@action`, `@permission_classes`, `@authentication_classes`, `@receiver`, `@shared_task`.

## HTTP Status Codes

200 OK · 201 created · 202 accepted · 204 no body · 400 invalid · 401 unauthenticated/challenge · 403 forbidden · 404 not found · 409 conflict · 429 throttle · 5xx server.

## Request/Response Flow

URL/router -> negotiation -> authentication -> permissions -> throttle -> handler -> serializer/ORM -> exception/response -> renderer.

## Serializer Flow

Input: instantiate -> validate -> `validated_data` -> save. Output: instance -> `.data` -> renderer.

## View/ViewSet Comparison

APIView: explicit HTTP methods. Generic views: reusable query/serializer and mixins. ViewSet: grouped resource actions; router maps URLs.

## Authentication Flow

Authenticator returns user/auth; permission decides access; queryset restricts visible objects.

## JWT Flow

Credentials -> access/refresh; bearer access -> protected API; refresh endpoint -> new access/rotated refresh. Signed != encrypted.

## Common Mistakes

- Object permission assumed to filter lists.
- `IsAuthenticated` assumed to enforce ownership.
- `bulk_create` assumed to call model save/signals.
- Transaction assumed to include broker/payment provider.

## Best Practices

- Explain trade-offs and failure modes.
- Enforce invariants in DB and application.
- Measure query plans and latency.
- Test multiple identities and negative paths.

## Interview Revision

`select_related` joins one-to-one/FK; `prefetch_related` batches many-valued relations. PUT replaces; PATCH partially updates. 202 means accepted, not complete.

## Memory Tricks

- **AuthN names; AuthZ permits.**
- **View routes work; serializer guards shape; DB guards truth.**

## One-Page Summary

Senior answers trace a request end to end, identify each layer's responsibility, state security/performance tradeoffs, and explain how tests and observability prove the design works.

# CRUD APIs Cheat Sheet

## Key Concepts

- CRUD: Create, Read, Update, Delete.
- Use resource-oriented plural URLs.
- Collection GET should be paginated, ordered, and scoped.
- Validate at serializer and database layers.

## Important Syntax

| Operation | HTTP | Common DRF view |
|---|---|---|
| List | GET collection | `ListAPIView` |
| Retrieve | GET detail | `RetrieveAPIView` |
| Create | POST collection | `CreateAPIView` |
| Replace | PUT detail | `UpdateAPIView` |
| Partial change | PATCH detail | `UpdateAPIView` |
| Delete | DELETE detail | `DestroyAPIView` |

## Frequently Used Classes and Methods

`ModelViewSet`, `ListSerializer(many=True)`, `get_queryset`, `perform_create`, `perform_update`, `perform_destroy`, `transaction.atomic`.

## Common Decorators

- `@action(detail=True, methods=["post"])` for a domain command.
- `@api_view(["GET", "POST"])` for compact function endpoints.

## HTTP Status Codes

201 created · 200 representation returned · 202 async accepted · 204 no content · 400 invalid · 401 unauthenticated · 403 forbidden · 404 missing · 409 conflict (contract-specific) · 429 throttled.

## Request/Response Flow

`HTTP intent -> router/view -> policy -> serializer/queryset -> ORM/DB -> status/body`.

## Serializer Flow

POST: `Serializer(data=...)`; PUT: `Serializer(instance, data=...)`; PATCH: add `partial=True`; validate before save.

## View/ViewSet Comparison

Use generic CRUD views for fixed narrow behavior; ModelViewSet for standard complete resource actions; custom action for workflow commands.

## Authentication Flow

Identity is not ownership. Set owner from `request.user`; scope `get_queryset()`.

## JWT Flow

Login -> access/refresh; access for resource requests; refresh only at refresh endpoint.

## Common Mistakes

- Treating PUT as PATCH.
- Unbounded list endpoints.
- Blind POST retry after timeout.
- Trusting client price, owner, or status.

## Best Practices

- Stable ordering plus pagination.
- Idempotency for retry-sensitive create.
- DB constraints for uniqueness.
- Explicit deletion and soft-delete policy.

## Interview Revision

POST is not idempotent by default; PUT is intended to replace; PATCH changes supplied fields; DELETE commonly returns 204.

## Memory Tricks

- **POST adds, GET reads, PUT replaces, PATCH tweaks, DELETE removes.**
- **Validate -> authorize -> persist -> represent.**

## One-Page Summary

CRUD is a public contract. Correct methods/statuses, scoped access, serializer validation, database constraints, pagination, and explicit retry/deletion semantics make it reliable.

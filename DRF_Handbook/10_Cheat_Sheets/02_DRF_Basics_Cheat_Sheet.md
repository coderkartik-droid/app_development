# DRF Basics Cheat Sheet

## Key Concepts

- `Request`: parsed API request with `.data`, `.query_params`, `.user`, `.auth`.
- `Response`: negotiated API response rendered after view execution.
- Parser: incoming bytes -> Python values.
- Renderer: Python primitives -> response representation.
- Content negotiation selects renderer from `Accept`.

## Important Syntax

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response

@api_view(["GET"])
def health(request):
    return Response({"status": "ok"})
```

## Frequently Used Classes and Methods

| Name | Purpose |
|---|---|
| `APIView` | Class-based API dispatch/policies |
| `Request.data` | Parsed body |
| `Request.query_params` | URL query values |
| `Response(data, status=...)` | API response |
| `ValidationError` | Structured invalid-input failure |
| `status.HTTP_201_CREATED` | Named status constant |

## Common Decorators

- `@api_view(["GET", "POST"])`
- `@permission_classes([...])`
- `@authentication_classes([...])`
- `@throttle_classes([...])`

## HTTP Status Codes

| Code | Meaning |
|---|---|
| 200 | Successful read/update |
| 201 | Created |
| 202 | Accepted, work pending |
| 204 | Success, no body |
| 400 | Invalid request |
| 401 | Missing/invalid authentication (scheme dependent) |
| 403 | Authenticated but disallowed |
| 404 | Not found/inaccessible |
| 429 | Throttled |

## Request/Response Flow

`URL -> DRF dispatch -> negotiation -> auth -> permissions -> throttle -> handler -> exception handling -> Response -> renderer`.

## Serializer Flow

Input: `Serializer(data=request.data) -> is_valid() -> validated_data -> save()`.
Output: `Serializer(instance).data -> primitive representation -> renderer`.

## View/ViewSet Comparison

| APIView | ViewSet |
|---|---|
| Method handlers (`get`, `post`) | Resource actions (`list`, `create`) |
| Explicit URL mapping | Often router-generated mapping |
| Maximum control | Reuses CRUD conventions |

## Authentication Flow

Authenticator establishes `request.user` and `request.auth`; permission decides whether request may proceed.

## JWT Flow

Obtain credentials -> access + refresh -> `Authorization: Bearer <access>` -> refresh endpoint when needed.

## Common Mistakes

- Returning Django `JsonResponse` when DRF `Response` behavior is needed.
- Reading GET filters from `request.data`.
- Assuming an HTTP client throws on 400/500.

## Best Practices

- Use named status constants.
- Keep API exceptions structured.
- Configure secure permission defaults.
- Document parser and renderer expectations.

## Interview Revision

**Parser reads; renderer writes.** Request data is parsed on demand; Response is rendered during finalization.

## Memory Tricks

- **DRF order:** Negotiate, Authenticate, Authorize, Throttle, Handle.
- **Request in, Response out.**

## One-Page Summary

DRF wraps Django with API-aware requests, responses, parsing, rendering, exceptions, authentication, and policy checks. The request lifecycle and status contract should be understood before implementing endpoint logic.

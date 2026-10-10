# Authentication Cheat Sheet

## Key Concepts

- Authentication identifies; authorization allows/denies.
- `request.user`: identity; `request.auth`: credential context.
- Permission checks are separate from authentication.
- Object checks do not automatically filter list results.

## Important Syntax

```python
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework_simplejwt.authentication.JWTAuthentication",
    ],
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
}
```

## Frequently Used Classes and Methods

| Class | Summary |
|---|---|
| `SessionAuthentication` | Django session; CSRF for unsafe methods |
| `BasicAuthentication` | Reusable password per request; HTTPS required |
| `TokenAuthentication` | DRF token record; default lifecycle limited |
| Simple JWT `JWTAuthentication` | Signed bearer access token |
| `IsAuthenticated` | Requires authenticated identity |
| `BasePermission` | Custom policy hooks |

## Common Decorators

- `@permission_classes([...])`
- `@authentication_classes([...])`
- `@throttle_classes([...])`

## HTTP Status Codes

401 commonly means credentials missing/invalid with challenge; 403 means policy denial; exact unauthenticated behavior depends on authentication scheme. 429 indicates throttling.

## Request/Response Flow

Client credential -> authenticator -> `request.user`/`request.auth` -> permission -> view -> response.

## Serializer Flow

Serializers validate input; trusted identity fields should be injected by view (`serializer.save(owner=request.user)`), not accepted from client.

## View/ViewSet Comparison

Authentication and permission classes apply to APIView, generic views, and ViewSets. Router generation does not secure routes.

## Authentication Flow

Session cookie -> server session; Basic header -> password verification; token header -> token lookup; JWT bearer -> signature/claim validation.

## JWT Flow

Obtain pair -> access token to APIs -> refresh token to refresh route -> rotate/revoke per policy. JWT is signed, not encrypted.

## Common Mistakes

- Authentication without queryset/object scope.
- Basic Base64 called encryption.
- Long-lived JWT with no revocation plan.
- Logging Authorization header.

## Best Practices

- HTTPS always.
- Secure cookies and CSRF for sessions.
- Short access lifetime and protected refresh token.
- Deny by default; test cross-user/tenant requests.

## Interview Revision

`has_permission` checks request/view; `has_object_permission` checks one retrieved object; list visibility belongs in queryset.

## Memory Tricks

- **AuthN = Name; AuthZ = Access.**
- **Access is short; refresh renews.**

## One-Page Summary

Select an authenticator for client type, then independently enforce permissions and scoped querysets. Protect credential storage, expiry, rotation, revocation, CSRF, TLS, and logs.

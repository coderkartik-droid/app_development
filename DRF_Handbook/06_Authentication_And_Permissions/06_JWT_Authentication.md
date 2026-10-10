# Introduction

JSON Web Token (JWT) authentication uses signed tokens containing claims. A client presents an access token; the API validates its signature and claims to establish identity.

# Why this concept exists

JWTs can support stateless verification across services, avoiding a database lookup for every access-token validation when the design and key management are sound.

# Real-world analogy

A JWT resembles a signed event pass with printed claims and a tamper-evident seal. The seal proves integrity; the pass is not necessarily secret or revocable by itself.

# Internal Working

A JWT has header, payload, and signature sections. The payload is Base64URL-encoded, not encrypted. The server verifies signature, algorithm, expiry, issuer/audience as configured, then authenticates the represented user.

# Authentication Flow (where applicable)

```text
credentials -> token issuer -> signed access/refresh tokens
client sends access token -> JWTAuthentication validates -> request.user
-> permissions -> view -> serializer/database -> response
```

# Syntax

```http
Authorization: Bearer <access-token>
```

# Complete Example 1

```python
# settings.py, using Simple JWT
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES": [
        "rest_framework_simplejwt.authentication.JWTAuthentication",
    ],
    "DEFAULT_PERMISSION_CLASSES": [
        "rest_framework.permissions.IsAuthenticated",
    ],
}
```

The authenticator validates bearer tokens, and the permission makes endpoints private by default. Install and configure a maintained package before enabling this class. Add public exceptions deliberately, such as login and health routes.

# Complete Example 2

```python
from rest_framework.views import APIView
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response

class AccountView(APIView):
    permission_classes = [IsAuthenticated]

    def get(self, request):
        return Response({"account_id": request.user.pk})
```

The authenticated user comes from the validated token, not from a request-body user ID. The view still requires authorization policy for individual account resources. Keep response claims minimal.

# Complete Example 3

```python
response = APIClient().get(
    "/api/account/",
    HTTP_AUTHORIZATION="Bearer expired-or-invalid-token",
)
assert response.status_code == 401
```

This is a negative-path test; configure JWTAuthentication and a consistent challenge scheme. Also test valid, expired, malformed, and wrong-signature tokens without exposing their contents in test logs.

# Advanced Example

Plan short access-token expiry, refresh rotation/blacklisting where appropriate, signing-key rotation, issuer/audience validation, and incident revocation. Stateless verification makes immediate revocation difficult unless a denylist/version check or short lifetime is used.

# Under the Hood

Signature verification proves the token was signed by a trusted key and was not modified. Claim validation checks temporal and contextual restrictions. A signed token's payload can be decoded by anyone who obtains it.

# Security Notes

Never put passwords, secrets, or sensitive personal data in JWT claims. Pin allowed algorithms, protect signing keys, use TLS, and avoid accepting unsigned or unexpected token types.

# Best Practices

- Use a vetted implementation and secure key storage.
- Keep access tokens short-lived.
- Validate expiry and intended audience/issuer.
- Define refresh, rotation, revocation, and logout semantics.

# Common Mistakes

- Treating JWT as encrypted data.
- Storing long-lived access tokens without protection.
- Assuming logout invalidates a copied stateless token.

# Debugging

Check header scheme, token format, signature/key, algorithm, expiry, clock skew, audience/issuer, and user activity.

# Interview Questions

1. What does JWT signature guarantee? Integrity/authenticity under trusted key, not confidentiality.
2. How can a stateless token be revoked? Use denylist/version state or short expiry with controlled refresh.
3. What belongs in a JWT? Minimal non-sensitive claims needed by the consumer.

# Practice Exercises

1. Configure JWT auth and protect a profile endpoint.
2. Test expiry and invalid signature behavior.
3. Design a key rotation and logout strategy.

# Chapter Summary

JWT is a signed claims format often used for bearer authentication. It moves credential state into a token but does not encrypt claims or solve revocation, authorization, and secure storage automatically.

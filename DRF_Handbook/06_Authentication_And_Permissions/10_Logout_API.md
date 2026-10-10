# Introduction

Logout ends an authenticated session from the client's perspective and, when supported, revokes the corresponding server-side credential. Stateless JWT logout needs an explicit revocation strategy.

# Why this concept exists

Users need to end access after signing out, losing a device, or detecting compromise.

# Real-world analogy

Returning a physical badge ends its use only if the building marks it invalid. Throwing away a copy does not disable a badge that someone else kept.

# Internal Working

Session logout invalidates server session state. JWT logout may blacklist a refresh token, revoke a server-side session record, or rely on short expiry. Removing a token only from the client is not server-side revocation.

# Authentication Flow (where applicable)

```text
client sends refresh/session credential -> logout endpoint
-> authenticate/authorize -> invalidate server-side credential -> response
```

# Syntax

```http
POST /api/logout/
{"refresh":"<refresh-token>"}
```

# Complete Example 1

```python
from rest_framework import serializers, status
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView
from rest_framework_simplejwt.tokens import RefreshToken

class LogoutInput(serializers.Serializer):
    refresh = serializers.CharField()

class LogoutView(APIView):
    permission_classes = [IsAuthenticated]

    def post(self, request):
        data = LogoutInput(data=request.data)
        data.is_valid(raise_exception=True)
        RefreshToken(data.validated_data["refresh"]).blacklist()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

This requires Simple JWT blacklist app and migrations. The serializer validates the input shape; `blacklist()` revokes the refresh token; 204 indicates no body. Production must ensure the token belongs to the authenticated user's session—JWT token validity alone may not establish that relation in every configuration. Catch only expected token errors and return a safe response.

# Complete Example 2

```python
from django.contrib.auth import logout
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from rest_framework.views import APIView

class SessionLogoutView(APIView):
    permission_classes = [IsAuthenticated]

    def post(self, request):
        logout(request)
        return Response(status=204)
```

Django's `logout` clears the current session. This example is for session authentication; unsafe session requests must pass CSRF validation. It does not revoke JWTs or other device sessions.

# Complete Example 3

```python
response = APIClient().post(
    "/api/logout/",
    {"refresh": refresh_token},
    format="json",
)
assert response.status_code == 204
```

After logout, test that the refresh token cannot obtain another access token. For session logout, verify the session no longer authenticates. Avoid accepting arbitrary tokens without binding/revocation checks.

# Advanced Example

Model device sessions explicitly when users need “log out this device” and “log out everywhere.” Store revocation state, rotate refresh tokens, and revoke sessions on password reset or account compromise according to policy.

# Under the Hood

Blacklisting adds server-side state to token validation. Session logout invalidates Django session state. Access JWTs already issued may remain usable until expiry unless the application also checks revocation/version state.

# Security Notes

Do not claim logout revokes every access token unless it does. Protect endpoint with authentication/CSRF as applicable, bind revocation to caller, and never log supplied tokens.

# Best Practices

- Explain logout guarantees to clients.
- Revoke refresh/session credentials server-side where required.
- Use short access lifetimes and rotation.
- Test reuse after logout.

# Common Mistakes

- Merely deleting a token from client memory and calling it revocation.
- Blacklisting a token without verifying its ownership/session context.
- Forgetting already-issued access tokens may survive.

# Debugging

Check blacklist app/migrations, token type, authenticated user, request body, CSRF, and follow-up refresh/access behavior.

# Interview Questions

1. Why is JWT logout not automatic? Stateless access tokens have no server session to clear.
2. What does refresh blacklisting achieve? Prevents further refresh with that credential.
3. Can access tokens survive logout? Often until expiry unless revocation checks exist.

# Practice Exercises

1. Implement logout for session auth and test session invalidation.
2. Configure refresh token blacklist and test reuse rejection.
3. Document the exact access-token logout guarantee.

# Chapter Summary

Logout behavior depends on the credential model. Revoke server-side state where required and communicate whether existing access tokens remain valid until expiry.

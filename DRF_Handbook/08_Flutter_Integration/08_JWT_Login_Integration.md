# Introduction

JWT login integration lets Flutter exchange credentials for access and refresh tokens, use access tokens on protected API requests, and renew them when they expire.

# Why this concept exists

The app needs to authenticate API calls without sending the user's password on every request.

# Real-world analogy

Login obtains a temporary entry pass plus a protected renewal voucher for later use.

# Internal Working

Flutter posts credentials to a token endpoint. DRF/Simple JWT verifies the password and signs tokens. Flutter stores them according to platform security and sends the access token as a bearer credential.

# Data Flow

Flutter login form -> POST credentials -> DRF token view -> Django auth -> token pair
-> secure storage -> Authorization header -> protected view -> response.

# Complete Example 1

Flutter login request:

```dart
Future<TokenPair> login(http.Client client, String username, String password) async {
  final response = await client.post(
    Uri.https('api.example.com', '/api/token/'),
    headers: {'Content-Type': 'application/json'},
    body: jsonEncode({'username': username, 'password': password}),
  );
  if (response.statusCode != 200) {
    throw StateError('Login failed');
  }
  final json = jsonDecode(response.body) as Map<String, dynamic>;
  return TokenPair(access: json['access'] as String,
                   refresh: json['refresh'] as String);
}
```

The request uses HTTPS and JSON, checks status, and parses access/refresh fields. Do not show raw server error details to users or log credentials/tokens. Use typed auth errors and secure storage instead of keeping tokens in ordinary preferences.

# Complete Example 2

DRF/Simple JWT setup:

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

The authenticator verifies access tokens; the permission protects views. Register token obtain and refresh URL routes separately. Public login routes must be available intentionally, with throttling and generic errors.

# Complete Example 3

Flutter bearer request:

```dart
Future<http.Response> getProfile(http.Client client, String access) {
  return client.get(
    Uri.https('api.example.com', '/api/profile/'),
    headers: {'Authorization': 'Bearer $access', 'Accept': 'application/json'},
  );
}
```

The access token belongs in the Authorization header, not URL or body. A 401 may trigger one coordinated refresh flow; avoid infinite retry loops and replace rotated credentials securely.

# Advanced Example

Create an auth repository that serializes refresh attempts, rotates refresh credentials, clears storage on failure, and handles logout/revocation limitations. Use secure storage supported by the platform and plan keychain/keystore behavior.

# Under the Hood

JWT is signed, not encrypted; Flutter can decode claims but must not treat decoded values as trusted authorization. The backend validates signature, type, expiry, and user state.

# Performance Notes

Avoid refreshing on every request; refresh only when needed and coalesce concurrent refresh calls.

# Security Notes

Use HTTPS, secure token storage, short access expiry, refresh rotation/revocation as needed, and redact tokens from diagnostics.

# Best Practices

- Never persist plaintext passwords.
- Keep token handling centralized.
- Treat server authorization as authoritative.
- Test expiration, refresh, logout, and invalid token paths.

# Common Mistakes

- Storing tokens in plain preferences.
- Sending refresh token to resource endpoints.
- Assuming logout instantly revokes every access JWT.

# Debugging

Check token endpoint, JSON keys, header scheme, secure-storage read/write, token expiry, server signing configuration, and clock skew.

# Interview Questions

1. Which JWT is sent to resource endpoints?
2. Is JWT payload encrypted?
3. Where should Flutter store refresh credentials?

# Practice Exercises

1. Build login and protected-profile flow.
2. Add secure storage and logout.
3. Implement single-flight refresh on expired access.

# Chapter Summary

JWT login is a credential lifecycle shared across both platforms. Flutter securely stores and sends tokens; DRF validates identity and independently authorizes every operation.

# Introduction

Authentication answers “who are you?” Authorization answers “what are you allowed to do?” APIs commonly authenticate a user or client with tokens sent in request headers.

# Why this concept exists

Servers must protect private data and actions. A mobile app can improve the experience, but the server must make the final access decision because client code can be modified.

# Real-world analogy

Authentication is showing a badge at a building entrance. Authorization is checking which rooms that badge opens.

# Internal Working

An authentication flow verifies a credential and issues a session/token. The client stores credentials appropriately and attaches them to protected requests. The server validates the credential, identifies a principal, then checks permission for each resource/action.

Never send passwords repeatedly to resource endpoints or embed privileged API secrets in the app. Use HTTPS and the service's documented OAuth/OIDC or authentication mechanism.

# Syntax

```dart
headers: {'authorization': 'Bearer $accessToken'}
```

# Complete Example 1

This app demonstrates an authenticated request shape using a clearly fake token and httpbin. It does not authenticate a real user. Add `http`.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const AuthBasicsApp());

class AuthBasicsApp extends StatelessWidget {
  const AuthBasicsApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: AuthBasicsPage());
}

class AuthBasicsPage extends StatefulWidget {
  const AuthBasicsPage({super.key});

  @override
  State<AuthBasicsPage> createState() => _AuthBasicsPageState();
}

class _AuthBasicsPageState extends State<AuthBasicsPage> {
  String _status = 'Not requested';

  Future<void> _callProtectedRoute() async {
    const fakeToken = 'demo-token-not-valid';
    try {
      final response = await http.get(
        Uri.https('httpbin.org', '/anything/private'),
        headers: {'authorization': 'Bearer $fakeToken'},
      );
      if (!mounted) return;
      setState(() => _status = 'HTTP ${response.statusCode} (echo demo only)');
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Request failed');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Authentication basics')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_status),
              FilledButton(
                onPressed: _callProtectedRoute,
                child: const Text('Send demo request'),
              ),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- The token is an explicit fake value, not a credential.
- `_callProtectedRoute` attaches an Authorization header using the Bearer scheme.
- httpbin echoes request structure; it does not authorize the route.
- The response status is displayed and transport errors are handled.
- `mounted` protects the asynchronous update.

### Widget tree

```text
MaterialApp → AuthBasicsPage → Scaffold → Center → Column(status, button)
```

### Execution flow

```text
Client attaches credential -> HTTPS request -> server validates token
                            <- status/response <- update UI
```

### ASCII UI preview

```text
+---------------------------+
| Authentication basics    |
+---------------------------+
| HTTP 200 (echo demo only) |
| [ Send demo request ]     |
+---------------------------+
```

### Example explanation

The demonstration proves how a header is attached, not how authentication works. A real protected server validates token signature/expiry/revocation and then authorization policy.

If the fake token is mistaken for a real secret, the design is unsafe. If the server trusts a user ID supplied by the client without checking the authenticated principal, data can be exposed.

**Common mistakes:** Keep passwords and privileged secrets out of source code, logs, and URLs.

# Complete Example 2

An authentication response may provide an access token and expiry:

```dart
final accessToken = response.accessToken;
final expiresAt = response.expiresAt;
```

Use the API's actual schema and store token material securely.

# Complete Example 3

Authorization should be enforced for each resource:

```text
authenticated principal + requested resource + policy -> allow/deny
```

Hiding a button is UX, not access control.

# Advanced Example

Use OAuth 2.0/OIDC flows suitable for native apps, generally authorization code with PKCE when applicable. Validate redirect URIs, state, nonce, issuer, audience, signature, and expiry according to the protocol/library. Prefer platform browser/system auth over embedded credential collection when required.

# Under the Hood

Tokens are credentials presented to the server. Bearer tokens grant access to whoever possesses them, so transport and storage protection matter. Server authorization maps validated identity and claims to permitted operations.

# Performance Notes

- Avoid repeated login prompts through controlled refresh.
- Cache token in memory after secure retrieval where appropriate.
- Do not make authorization decisions based on client-only state.

# Best Practices

- Use established auth protocols and libraries.
- Use HTTPS and secure storage.
- Validate authorization on every server operation.
- Redact tokens.
- Handle expiry, revocation, and sign-out.

# Common Mistakes

- Confusing authentication and authorization.
- Storing credentials in plain preferences.
- Hard-coding privileged API secrets.
- Trusting client-provided user IDs.
- Logging bearer tokens.

# Debugging

- Check token presence/expiry without printing token.
- Inspect 401 versus 403 responses.
- Verify issuer/audience/scopes on server.
- Test revoked and expired sessions.

# Interview Questions

1. Authentication vs authorization? Identity verification vs permission decision.
2. Why is a bearer token sensitive? Whoever possesses it may use its authority.
3. Why HTTPS? Protects credential transport.
4. Is hiding a UI action authorization? No.

# Try It Yourself

1. Draw login, token, protected request, expiry, and sign-out.
2. Map 401 and 403 to separate UX.
3. Identify secrets that must never ship in app binary.
4. Describe authorization enforcement on backend.

# Challenge Project

Design a secure session flow with OAuth/OIDC, secure token storage, refresh, expiry, account-scoped data cleanup, and server authorization.

# Quick Revision Sheet

- Authentication = identity; authorization = access.
- Use established protocols and HTTPS.
- Bearer tokens are secrets.
- Store securely; validate server-side.
- Handle expiry, revocation, and sign-out.

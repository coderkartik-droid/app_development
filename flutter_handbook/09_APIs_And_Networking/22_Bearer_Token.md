# Introduction

A bearer token is a credential that grants access to whoever presents it. APIs commonly carry it in the HTTP `Authorization` header as `Bearer <token>`.

# Why this concept exists

After a user signs in, the app can make authorized requests without sending the user's password on every call.

# Real-world analogy

A bearer token resembles a temporary access pass. Possession is enough to present it, so losing the pass is dangerous.

# Internal Working

The client obtains a token through an authentication flow, stores it according to sensitivity and lifetime, and attaches it to protected requests over HTTPS. The server validates signature/introspection, expiry, audience, issuer, and scopes as applicable.

# Syntax

```dart
headers: {'authorization': 'Bearer $token'}
```

# Complete Example 1

This app constructs a request with a fake bearer token and displays the server response status. Add `http`. The echo endpoint does not validate credentials.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const BearerTokenApp());

class BearerTokenApp extends StatelessWidget {
  const BearerTokenApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: BearerTokenPage());
}

class BearerTokenPage extends StatefulWidget {
  const BearerTokenPage({super.key});

  @override
  State<BearerTokenPage> createState() => _BearerTokenPageState();
}

class _BearerTokenPageState extends State<BearerTokenPage> {
  String _status = 'Ready';

  Future<void> _request() async {
    const token = 'fake-example-token';
    try {
      final response = await http.get(
        Uri.https('httpbin.org', '/anything/resource'),
        headers: {'authorization': 'Bearer $token'},
      );
      if (!mounted) return;
      setState(() => _status = 'Response: ${response.statusCode}');
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Network failure');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Bearer header')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_status),
              FilledButton(onPressed: _request, child: const Text('Call API')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- The token literal is explicitly fake for demonstration.
- `_request` builds an HTTPS URI and attaches the standard Authorization header.
- httpbin echoes the request rather than checking the token.
- Status and transport failure are displayed distinctly.
- `mounted` guards UI updates.

### Widget tree

```text
MaterialApp → BearerTokenPage → Scaffold → Center → Column
                                                  ├→ status
                                                  └→ Call API
```

### Execution flow

```text
Token provider -> Authorization header -> HTTPS request
               -> server validation -> response -> UI
```

### ASCII UI preview

```text
+---------------------------+
|       Bearer header       |
+---------------------------+
| Response: 200             |
| [       Call API       ]  |
+---------------------------+
```

### Example explanation

`Bearer` is the scheme name and the token follows it. The server must validate the credential. An echo service returning 200 says nothing about token validity.

If HTTPS is removed, the token can be exposed in transit. If diagnostics print all headers, the token can leak into logs.

**Common mistakes:** Never put bearer tokens in query parameters or screenshots.

# Complete Example 2

Add a token per request through a service method:

```dart
Future<http.Response> getPrivate(Uri uri, String token) {
  return client.get(uri, headers: {'authorization': 'Bearer $token'});
}
```

For production, the token should come from a controlled session provider, not arbitrary UI input.

# Complete Example 3

On 401, refresh once according to auth policy and retry only if safe:

```text
request -> 401 -> refresh token -> retry original request once
```

Prevent infinite refresh loops and coordinate concurrent refresh calls.

# Advanced Example

Use an interceptor/client wrapper that attaches current token, redacts headers, handles refresh single-flight, and retries eligible requests. Do not retry non-idempotent operations without idempotency protection.

# Under the Hood

The Authorization header is transmitted as HTTP metadata. TLS protects it in transit; token validation is server-side. Secure storage protects at rest but cannot prevent a compromised running process from accessing a token in memory.

# Performance Notes

- Avoid secure-storage reads on every request; use a managed in-memory session cache.
- Refresh before expiry when appropriate.
- Avoid duplicated concurrent refreshes.

# Best Practices

- Use short-lived access tokens where supported.
- Store refresh credentials securely.
- Redact tokens from logs.
- Validate audience, issuer, expiry, and scope server-side.
- Clear tokens on sign-out.

# Common Mistakes

- Missing `Bearer ` prefix.
- Sending a token to an untrusted host.
- Repeatedly refreshing in a loop.
- Retrying unsafe writes automatically.
- Treating 200 from an echo endpoint as auth success.

# Debugging

- Confirm header name and scheme without exposing value.
- Inspect 401/403 and server auth logs via request ID.
- Check clock skew and expiry.
- Verify token audience/scope.

# Interview Questions

1. What is a bearer credential? Access is granted to its possessor.
2. Where should it be sent? Usually Authorization header over HTTPS.
3. Why redact it? It can be replayed by anyone who obtains it.
4. What is a refresh loop? Repeated refresh/retry cycle that never resolves auth failure.

# Try It Yourself

1. Build a fake token provider interface.
2. Redact authorization in request logging.
3. Design one-refresh retry logic.
4. Test 401 and 403 separately.

# Challenge Project

Implement a mock authenticated API client with secure-token interface, one-time refresh, concurrency control, and safe retry rules.

# Quick Revision Sheet

- `Authorization: Bearer token`.
- Bearer possession grants use; protect it.
- HTTPS in transit, secure storage at rest.
- Redact logs and validate server-side.
- Refresh once with loop protection.

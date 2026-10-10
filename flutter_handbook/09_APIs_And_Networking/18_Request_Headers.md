# Introduction

HTTP headers carry metadata about a request or response. They describe content format, authentication, caching, tracing, and client preferences.

# Why this concept exists

Headers keep protocol metadata separate from the resource body and URI. Servers use them to interpret and authorize requests.

# Real-world analogy

Headers are labels on a parcel: delivery instructions, sender credentials, and handling notes. They do not replace the package contents.

# Internal Working

The client sends header name/value pairs with the request. Header names are case-insensitive in HTTP. Sensitive headers such as Authorization must be protected from logs and should only be sent over HTTPS.

# Syntax

```dart
headers: {
  'accept': 'application/json',
  'content-type': 'application/json',
  'authorization': 'Bearer $token',
}
```

# Complete Example 1

This app sends an `Accept` header and displays the response content type. Add `http`.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const HeadersApp());

class HeadersApp extends StatelessWidget {
  const HeadersApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: HeadersPage());
}

class HeadersPage extends StatefulWidget {
  const HeadersPage({super.key});

  @override
  State<HeadersPage> createState() => _HeadersPageState();
}

class _HeadersPageState extends State<HeadersPage> {
  String _result = 'Ready';

  Future<void> _send() async {
    try {
      final response = await http.get(
        Uri.https('jsonplaceholder.typicode.com', '/posts/1'),
        headers: {'accept': 'application/json'},
      );
      if (!mounted) return;
      setState(() => _result =
          'HTTP ${response.statusCode}\n${response.headers['content-type'] ?? 'No content type'}');
    } catch (error) {
      if (!mounted) return;
      setState(() => _result = 'Request failed');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Request headers')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_result, textAlign: TextAlign.center),
              FilledButton(onPressed: _send, child: const Text('Send')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- The GET includes `Accept: application/json`, requesting a JSON representation.
- The response status and content type are read from the response.
- Exceptions are handled separately from HTTP response codes.
- `mounted` protects the update.
- A button triggers the request.

### Widget tree

```text
MaterialApp → HeadersPage → Scaffold → Center → Column(Text, Button)
```

### Execution flow

```text
Set request headers -> send HTTP -> server negotiates/authorizes
                    <- response headers/body <- display metadata
```

### ASCII UI preview

```text
+---------------------------+
|      Request headers      |
+---------------------------+
| HTTP 200                  |
| application/json; charset |
| [          Send         ] |
+---------------------------+
```

### Example explanation

`Accept` describes the desired response format; it does not describe the request body. For a JSON POST, use `Content-Type` too. The example does not log or transmit credentials.

If `Accept` is omitted, server defaults apply. If Authorization is logged for debugging, a valid token may leak.

**Common mistakes:** Do not put tokens in query parameters or send them over plain HTTP.

# Complete Example 2

Set JSON body format:

```dart
headers: {'content-type': 'application/json'}
```

This belongs on requests with JSON bodies; it is not a substitute for encoding the body.

# Complete Example 3

Attach a request ID:

```dart
headers: {'x-request-id': requestId}
```

Use a unique identifier for tracing when the API supports it. Do not reuse a request ID for unrelated operations.

# Advanced Example

Centralize default headers in an API client, then add per-request headers where necessary. Token refresh should update the authorization header safely. Avoid mutable global maps shared across concurrent requests.

# Under the Hood

Headers are serialized as HTTP metadata. Servers, proxies, and caches can use them for content negotiation, authentication, caching, and tracing. Header values may have size and character constraints.

# Performance Notes

- Keep headers small.
- Avoid repeatedly encoding large metadata.
- Reuse shared defaults.
- Never attach unnecessary sensitive values.

# Best Practices

- Distinguish `Accept` and `Content-Type`.
- Use HTTPS for Authorization.
- Redact secrets in logs.
- Use standard header names when possible.
- Follow API header contract.

# Common Mistakes

- Wrong content type.
- Missing bearer prefix.
- Case-sensitive header lookup assumptions.
- Logging authorization tokens.
- Storing mutable global headers that race.

# Debugging

- Inspect sanitized outgoing header names/values.
- Verify proxy or CORS behavior on web.
- Check content type and accept negotiation.
- Confirm server expects exact custom header.

# Interview Questions

1. What does `Accept` indicate? Desired response representation.
2. What does `Content-Type` indicate? Format of the sent body.
3. Are header names case-sensitive? HTTP field names are case-insensitive.
4. Why redact Authorization? It contains credentials.

# Try It Yourself

1. Add `Content-Type` to a JSON POST.
2. Add a request ID and server correlation.
3. Add conditional cache headers.
4. Review logs for sensitive headers.

# Challenge Project

Create a shared HTTP client that adds JSON defaults and request IDs, attaches bearer tokens per request, and redacts sensitive headers in diagnostics.

# Quick Revision Sheet

- Headers carry metadata.
- `Accept` = desired response; `Content-Type` = body format.
- Authorization belongs over HTTPS and must be redacted.
- Centralize defaults; keep per-request values scoped.

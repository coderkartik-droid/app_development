# Introduction

API best practices make networking code secure, testable, resilient, and understandable. The right approach considers both Flutter code and the server contract.

# Why this concept exists

Network boundaries are unreliable and exposed. A feature that works on a fast development network may fail under latency, server errors, expired sessions, or malformed responses.

# Real-world analogy

A dependable delivery service labels parcels clearly, checks identity, tracks progress, handles delays, and has a recovery plan when something goes wrong.

# Internal Working

Production API path:

```text
UI -> state -> repository -> configured client -> HTTPS server
UI <- state <- typed result/error <- decoded response <-
```

Each layer should have a responsibility. The API client handles transport; repository coordinates sources/policy; state holder models presentation; UI renders.

# Syntax

```dart
final response = await client.get(
  Uri.https(host, path, queryParameters),
).timeout(const Duration(seconds: 10));
```

# Complete Example 1

This runnable example demonstrates a centralized client with safe URI construction, timeout, status checking, and typed result mapping. Add `http`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class ApiItem {
  const ApiItem(this.title);
  final String title;
}

class ApiClient {
  ApiClient(this.client);
  final http.Client client;

  Future<ApiItem> getItem(int id) async {
    final response = await client
        .get(Uri.https('jsonplaceholder.typicode.com', '/posts/$id'))
        .timeout(const Duration(seconds: 8));
    if (response.statusCode != 200) {
      throw Exception('Unexpected HTTP status ${response.statusCode}');
    }
    final json = jsonDecode(response.body) as Map<String, dynamic>;
    final title = json['title'];
    if (title is! String) throw const FormatException('title must be a string');
    return ApiItem(title);
  }
}

void main() => runApp(const BestApiApp());

class BestApiApp extends StatelessWidget {
  const BestApiApp({super.key});

  @override
  Widget build(BuildContext context) =>
      MaterialApp(home: BestApiPage(api: ApiClient(http.Client())));
}

class BestApiPage extends StatefulWidget {
  const BestApiPage({required this.api, super.key});
  final ApiClient api;

  @override
  State<BestApiPage> createState() => _BestApiPageState();
}

class _BestApiPageState extends State<BestApiPage> {
  String _text = 'Tap load';

  Future<void> _load() async {
    setState(() => _text = 'Loading...');
    try {
      final item = await widget.api.getItem(1);
      if (mounted) setState(() => _text = item.title);
    } catch (error) {
      if (mounted) setState(() => _text = 'Could not load item');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('API best practices')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_text),
              FilledButton(onPressed: _load, child: const Text('Load')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `ApiClient` is injected with an HTTP client.
- `getItem` constructs the URI safely, applies a timeout, validates status, decodes JSON, and validates the required title.
- `ApiItem` exposes typed data.
- The page uses the client and displays loading, success, or a friendly error.
- The injected client can be replaced with a mock in tests.

### Widget tree

```text
BestApiApp → BestApiPage → Scaffold → Column(Text, Load)
```

### Execution flow

```text
UI -> ApiClient -> HTTPS -> status/JSON validation -> ApiItem -> UI
```

### ASCII UI preview

```text
+---------------------------+
|   API best practices      |
+---------------------------+
| sunt aut facere...        |
| [          Load         ] |
+---------------------------+
```

### Example explanation

This centralizes request construction and validates response fields at the boundary. A production app should add a typed error taxonomy, close the client at its owner lifecycle, and use repository/state abstractions.

If validation is removed, invalid data can reach UI. If timeout is removed, a request can wait too long. If the client is not injected, tests become harder.

**Common mistakes:** Do not hide all diagnostics behind one generic exception in service code; translate for UI while preserving causes internally.

# Complete Example 2

Inject auth per request and redact logs:

```dart
headers: {'authorization': 'Bearer $accessToken'}
```

Never print that header.

# Complete Example 3

Retry policy should include maximum attempts, backoff, jitter, and idempotency checks. Do not retry every request or status.

# Advanced Example

Create a shared API foundation: base URI/configuration, client lifecycle, interceptors, typed DTOs/errors, correlation IDs, auth refresh, cancellation, logging redaction, mock tests, contract tests, and observability.

# Under the Hood

The networking package handles asynchronous transport; Flutter receives futures/streams; response bytes are decoded and mapped. UI rebuilds after state changes. Server-side validation and authorization remain authoritative.

# Performance Notes

- Reuse clients.
- Paginate and cache appropriately.
- Prevent duplicate requests.
- Keep JSON parsing off frame-critical paths when large.
- Measure real device behavior.

# Best Practices

- HTTPS and URI builders.
- Typed DTOs and errors.
- Timeouts and bounded retries.
- Dependency injection.
- Redacted structured logs.
- Server-side authorization and validation.

# Common Mistakes

- HTTP in widget build.
- Secrets in app source or logs.
- No timeout or status handling.
- Blind retries.
- No pagination or stale response protection.

# Debugging

- Capture method/path/status/duration/request ID.
- Separate transport, HTTP, parse, and domain failures.
- Reproduce with mocks and target platform.
- Check authentication and environment.

# Interview Questions

1. What belongs in an API client vs repository?
2. How do you retry safely?
3. How do you protect tokens?
4. How do you prevent stale search results?
5. How do you test malformed API responses?

# Try It Yourself

1. Add a mock-client test.
2. Add typed status errors.
3. Add auth header redaction.
4. Implement a safe retry policy for GET.

# Challenge Project

Create an API foundation package with injectable client, base URL, auth provider, timeout, typed errors, JSON converters, and tests.

# Quick Revision Sheet

- Separate UI, state, repository, and client.
- Use HTTPS, typed models, and URI builders.
- Validate status and body.
- Set timeout; retry only safely.
- Protect credentials and test through injection.

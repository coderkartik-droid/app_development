# Introduction

The `http` package is a cross-platform Dart package for sending HTTP requests. It provides convenient top-level functions for simple calls and an `http.Client` for reusable request sessions and testable APIs.

# Why this concept exists

Flutter's widget library does not include a complete HTTP API. The package provides a consistent interface for mobile, desktop, and web while keeping request code separate from UI.

# Real-world analogy

The package is a courier service. The app prepares an addressed request, the client carries it to the server, and a response object brings status, headers, and body back.

# Internal Working

Top-level methods use a short-lived client for one call. For a series of requests, a reusable `Client` can reuse connections and should be closed by its owner. Browser restrictions, TLS, proxy behavior, and platform support can differ.

# Syntax

```dart
final client = http.Client();
try {
  final response = await client.get(uri);
} finally {
  client.close();
}
```

# Complete Example 1

This app uses a reusable client to fetch one public post. Add `http` to dependencies.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const HttpPackageApp());

class HttpPackageApp extends StatelessWidget {
  const HttpPackageApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: HttpPackagePage());
}

class HttpPackagePage extends StatefulWidget {
  const HttpPackagePage({super.key});

  @override
  State<HttpPackagePage> createState() => _HttpPackagePageState();
}

class _HttpPackagePageState extends State<HttpPackagePage> {
  late final http.Client _client;
  String _text = 'Tap to load';

  @override
  void initState() {
    super.initState();
    _client = http.Client();
  }

  Future<void> _load() async {
    setState(() => _text = 'Loading...');
    try {
      final response = await _client.get(
        Uri.https('jsonplaceholder.typicode.com', '/posts/1'),
      );
      if (response.statusCode != 200) {
        throw http.ClientException('HTTP ${response.statusCode}');
      }
      final json = jsonDecode(response.body) as Map<String, dynamic>;
      if (!mounted) return;
      setState(() => _text = json['title'] as String);
    } catch (error) {
      if (!mounted) return;
      setState(() => _text = 'Request failed');
    }
  }

  @override
  void dispose() {
    _client.close();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('http package')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_text, textAlign: TextAlign.center),
              FilledButton(onPressed: _load, child: const Text('GET')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- The imports provide JSON decoding, Material widgets, and HTTP client classes.
- `_client` is created once in `initState` and reused.
- `_load` displays a loading message, performs GET, validates status, and decodes the title.
- `ClientException` marks an unexpected HTTP status for this sample.
- `mounted` protects state after the async gap.
- `dispose` closes the client it owns.
- The UI exposes a button and result text.

### Widget tree

```text
MaterialApp -> HttpPackagePage -> Scaffold -> Center -> Column(Text, Button)
```

### Execution flow

```text
Button -> reusable Client.get -> response -> status check -> JSON decode
       -> setState -> displayed title
Page disposed -> client.close
```

### ASCII UI preview

```text
+---------------------------+
|        http package       |
+---------------------------+
| sunt aut facere...        |
| [          GET          ] |
+---------------------------+
```

### Example explanation

The reusable client belongs to the page and is closed when the page ends. In a larger app, an app-scoped API client/repository may own it instead. If the client is never closed, resources can remain active. If a global top-level function is used for every call, connection reuse/configuration is less explicit.

**Common mistakes:** Do not create/close a shared client inside each widget build. Define one clear owner.

**Professional use case:** A configured API client shared by repositories, with mock clients injected in tests.

# Complete Example 2

Send JSON using the package:

```dart
final response = await client.post(
  uri,
  headers: {'content-type': 'application/json'},
  body: jsonEncode({'title': title}),
);
```

Use the endpoint's expected schema and validate its response.

# Complete Example 3

Inject a client for testing:

```dart
class ApiService {
  ApiService(this.client);
  final http.Client client;
}
```

Tests can pass a mock client to produce deterministic status/body combinations.

# Advanced Example

Wrap or subclass `http.BaseClient` for shared headers, tracing, or test interception. Avoid logging authorization headers. Handle cancellation separately; not every top-level call supports the same cancellation mechanism.

# Under the Hood

The package adapts Dart HTTP abstractions to platform/runtime networking. Responses expose decoded text through `body` or raw bytes through `bodyBytes`. The package does not automatically map JSON to domain models or enforce API-specific success.

# Performance Notes

- Reuse clients.
- Stream large transfers rather than buffering all bytes.
- Set timeouts.
- Avoid duplicate requests.
- Close owned clients.

# Best Practices

- Centralize client configuration.
- Inject clients for tests.
- Use `Uri` builders.
- Check status and content type.
- Keep errors and secrets sanitized.

# Common Mistakes

- Forgetting to close a long-lived client.
- Using response body when endpoint returns binary bytes.
- Treating all non-2xx responses as thrown automatically.
- Hard-coding tokens in a global header.

# Debugging

- Inspect request method/URI and response status.
- Check client lifecycle and mock setup.
- Verify platform network permission/configuration.
- Distinguish body decoding from transport.

# Interview Questions

1. Why use `http.Client`? Reuse/configuration and testability across multiple requests.
2. Why close a client? Release underlying resources and connections.
3. What are `body` and `bodyBytes`? Text and raw byte response representations.
4. Does package success imply HTTP 2xx? No; inspect status.

# Try It Yourself

1. Add common `Accept` headers.
2. Inject a mock client into a service.
3. Add a timeout and inspect behavior.
4. Download a small binary response through `bodyBytes`.

# Challenge Project

Build an API client wrapper with base URL, common headers, timeout, injectable client, and typed error mapping.

# Quick Revision Sheet

- Add `http` dependency and import it.
- Use reusable `Client` for multi-call services.
- Close clients at their owner lifecycle.
- Check status and decode intentionally.
- Inject clients for tests.

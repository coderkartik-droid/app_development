# Introduction

API interviews evaluate how you design and operate network features, not only how you call `http.get`. Strong answers explain request lifecycle, error states, security, data modeling, and testability.

# Why this concept exists

Networking crosses application, platform, and server boundaries. A developer needs to reason about latency, failures, schema changes, and authorization.

# Real-world analogy

An API feature is a supply chain: an order is placed, routed, checked, fulfilled, delivered, and reconciled. An answer should describe the entire chain.

# Internal Working

Typical flow: widget action → state manager → repository → HTTP client → server → response → decode/model → state → widgets. Discuss the lifecycle and failure path at every boundary.

# Syntax

```dart
final result = await repository.loadItems();
```

The UI should consume an app-level operation, not know transport details unless the feature is very small.

# Complete Example 1

This runnable example illustrates a typed service and visible async state. Add `http`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class Item {
  const Item(this.title);
  final String title;
}

class ItemApi {
  ItemApi(this.client);
  final http.Client client;

  Future<Item> fetch() async {
    final response = await client.get(
      Uri.https('jsonplaceholder.typicode.com', '/posts/1'),
    ).timeout(const Duration(seconds: 8));
    if (response.statusCode != 200) {
      throw Exception('HTTP ${response.statusCode}');
    }
    final json = jsonDecode(response.body) as Map<String, dynamic>;
    return Item(json['title'] as String);
  }
}

void main() => runApp(const ApiInterviewApp());

class ApiInterviewApp extends StatelessWidget {
  const ApiInterviewApp({super.key});

  @override
  Widget build(BuildContext context) =>
      MaterialApp(home: ApiInterviewPage(api: ItemApi(http.Client())));
}

class ApiInterviewPage extends StatefulWidget {
  const ApiInterviewPage({required this.api, super.key});
  final ItemApi api;

  @override
  State<ApiInterviewPage> createState() => _ApiInterviewPageState();
}

class _ApiInterviewPageState extends State<ApiInterviewPage> {
  String _message = 'Tap load';

  Future<void> _load() async {
    setState(() => _message = 'Loading');
    try {
      final item = await widget.api.fetch();
      if (mounted) setState(() => _message = item.title);
    } catch (error) {
      if (mounted) setState(() => _message = 'Could not load item');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('API interview example')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_message),
              FilledButton(onPressed: _load, child: const Text('Load')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `Item` is the typed app model.
- `ItemApi` owns HTTP, timeout, status validation, JSON decoding, and mapping.
- The page receives the API dependency rather than constructing requests in build.
- `_load` models loading, success, and user-facing failure.
- The button starts the operation.

### Widget tree

```text
MaterialApp → ApiInterviewPage → Scaffold → Column(message, Load)
```

### Execution flow

```text
Tap -> API client -> HTTPS -> status/JSON/model -> page state -> UI
                         \-> exception -> error state
```

### ASCII UI preview

```text
+---------------------------+
| API interview example     |
+---------------------------+
| sunt aut facere...        |
| [          Load         ] |
+---------------------------+
```

### Example explanation

This sample combines API and page to stay compact; in production, a repository/state layer and explicit client ownership improve testability. A complete answer should mention how the client is closed and errors are typed.

If the request is made from `build`, it may repeat. If status checking is removed, an error payload can be parsed as an item.

**Common mistakes:** Avoid claiming all non-200 statuses are transport failures.

# Complete Example 2

Test a client with a mock:

```dart
final client = MockClient((request) async =>
    http.Response('{"title":"Hello"}', 200));
```

Assert request URI/method and parsed value.

# Complete Example 3

Authentication lifecycle:

```text
secure refresh token -> obtain access token -> attach bearer -> handle expiry
```

Never use a local boolean as authorization.

# Advanced Example

## Interview prompts and concise answers

1. **How do you prevent duplicate calls?** Keep futures outside build, guard in-flight work, and deduplicate by request key.
2. **How do you handle errors?** Separate transport, status, parse, and domain failures.
3. **How do you secure tokens?** HTTPS, secure storage, short lifetimes, rotation, redaction.
4. **How do you support offline?** Local source of truth, durable outbox, idempotent retries, conflict policy.
5. **How do you evolve models?** Backward-compatible parsing and tested schema/API versions.
6. **How do you test?** Mock HTTP for service logic; repository/state tests; integration tests for critical flows.

# Under the Hood

HTTP work is asynchronous. Responses are decoded and mapped before state emits a new value. Widget rebuilds then reflect the state. Any layer can fail, and errors should retain causal information.

# Performance Notes

- Discuss pagination, caching, connection reuse, and payload size.
- Profile before asserting a bottleneck.
- Keep parsing and large transformations off frame-critical work.

# Best Practices

- State assumptions and endpoint contract.
- Separate responsibilities.
- Handle loading/empty/error/success.
- Protect credentials and user data.
- Include retry, cancellation, and tests where appropriate.

# Common Mistakes

- Naming libraries without explaining design.
- Ignoring non-2xx responses.
- No timeout or retry policy.
- Treating connectivity as server reachability.
- Storing secrets in source code.

# Debugging

- Trace one request from input to UI.
- Inspect sanitized request/response metadata.
- Reproduce with mock and target runtime.
- Test cold start and offline paths.

# Interview Questions

1. Describe the full request-response lifecycle.
2. How are PUT and PATCH different?
3. How do you prevent stale search results?
4. How do you design an offline-first mutation?
5. What are REST/GraphQL trade-offs?
6. Why use repositories and DTOs?
7. How do bearer and refresh tokens work?
8. How do you debug TLS versus HTTP errors?
9. How do you test API parsing?
10. How do you safely retry a POST?

# Try It Yourself

1. Explain a networking feature in two minutes.
2. Draw its layers and error path.
3. Write tests for malformed JSON and 401.
4. Identify retry-safe methods.

# Challenge Project

Present and implement an API feature including architecture, models, state, security, pagination/cache policy, error handling, and tests.

# Quick Revision Sheet

- Request lifecycle spans UI to server and back.
- Distinguish transport/status/parse/domain failures.
- Secure credentials and use HTTPS.
- Use typed models and repositories.
- Test retries, stale results, and offline flows.

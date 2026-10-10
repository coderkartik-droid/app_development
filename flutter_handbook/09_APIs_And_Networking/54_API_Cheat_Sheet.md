# Introduction

This cheat sheet summarizes core API and networking concepts for Flutter. Use it to recall the request lifecycle and select the right building block; refer to the detailed chapters for implementation and trade-offs.

# Why this concept exists

Networking spans URI construction, transport, status handling, parsing, state, security, and lifecycle. A compact map helps avoid forgetting a critical stage.

# Real-world analogy

This is a route card for a delivery: address, parcel type, route, receipt, and exception procedure.

# Internal Working

```text
User intent -> state -> repository -> HTTP client
            -> DNS/TLS/HTTP -> server -> status/headers/body
            -> decode/validate/model -> state -> Flutter widgets
```

# Syntax

## HTTP request

```dart
final uri = Uri.https('api.example.com', '/v1/items', {'limit': '20'});
final response = await client.get(uri).timeout(const Duration(seconds: 10));
```

## JSON

```dart
final map = jsonDecode(response.body) as Map<String, dynamic>;
final model = Item.fromJson(map);
```

# Complete Example 1

The following minimal runnable Flutter screen sends a GET, checks status, decodes JSON, and displays a typed title. Add `http`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const ApiCheatSheetApp());

class ApiCheatSheetApp extends StatelessWidget {
  const ApiCheatSheetApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: CheatSheetPage());
}

class CheatSheetPage extends StatefulWidget {
  const CheatSheetPage({super.key});

  @override
  State<CheatSheetPage> createState() => _CheatSheetPageState();
}

class _CheatSheetPageState extends State<CheatSheetPage> {
  String _message = 'Press load';
  bool _loading = false;

  Future<void> _load() async {
    setState(() => _loading = true);
    try {
      final response = await http
          .get(Uri.https('jsonplaceholder.typicode.com', '/posts/1'))
          .timeout(const Duration(seconds: 8));
      if (response.statusCode != 200) {
        throw Exception('HTTP ${response.statusCode}');
      }
      final json = jsonDecode(response.body) as Map<String, dynamic>;
      final title = json['title'];
      if (title is! String) throw const FormatException('Invalid title');
      if (mounted) setState(() => _message = title);
    } catch (error) {
      if (mounted) setState(() => _message = 'Request failed');
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('API cheat sheet')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_message, textAlign: TextAlign.center),
              FilledButton(
                onPressed: _loading ? null : _load,
                child: const Text('Load'),
              ),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- Imports provide JSON decoding, widgets, and HTTP calls.
- `_load` starts loading, creates a safe HTTPS URI, applies a timeout, and sends GET.
- A non-200 response is rejected before parsing.
- JSON shape and title type are checked.
- The UI updates only while mounted.
- `finally` clears loading even if transport, status, or parsing fails.
- The button is disabled during the request.

### Widget tree

```text
MaterialApp → CheatSheetPage → Scaffold → Center → Column(message, button)
```

### Execution flow

```text
Tap -> GET -> status -> JSON -> type validation -> state -> UI
            \-> timeout/error -> failure state
```

### ASCII UI preview

```text
+---------------------------+
|     API cheat sheet       |
+---------------------------+
| sunt aut facere...        |
| [          Load         ] |
+---------------------------+
```

### Example explanation

This minimal pattern includes the stages that are easiest to forget: safe URI, timeout, status check, parse validation, lifecycle guard, and loading cleanup. Production code should move the request into an injected API client/repository and use typed errors.

If any stage is removed, the corresponding failure becomes harder to handle: no status check can parse error bodies, no timeout can wait indefinitely, and no `finally` can leave the UI stuck loading.

**Common mistakes:** Do not copy demo endpoint URLs or API keys into production without checking the provider contract.

# Complete Example 2

Method quick reference:

```text
GET    read
POST   create/process
PUT    replace
PATCH  partial update
DELETE remove
```

Use the endpoint's actual semantics and retry rules.

# Complete Example 3

Security quick reference:

```text
HTTPS + secure token storage + server authorization + redacted logs
```

No client-side check replaces server authorization.

# Advanced Example

## Fast design checklist

- Data shape and sensitivity classified.
- API method, URI, headers, body, and response contract documented.
- Typed DTO and validation implemented.
- Loading, empty, error, timeout, and retry states handled.
- Pagination/cache/stale-result policy defined.
- Auth refresh and sign-out lifecycle tested.
- Client/repository injected and mock-tested.
- Logs sanitized and request IDs available.

# Under the Hood

HTTP libraries send asynchronous protocol data through the runtime/platform network stack. The response is status/headers/body; JSON decoding and model mapping happen in Dart. State updates then trigger Flutter rendering.

# Performance Notes

- Reuse clients.
- Paginate large data.
- Cache with explicit freshness.
- Debounce search.
- Stream large files.
- Profile parsing and rebuilds.

# Best Practices

- Use HTTPS and URI builders.
- Check status and validate payloads.
- Keep networking out of build.
- Separate layers and inject dependencies.
- Time out, retry safely, and redact secrets.
- Test cold, offline, error, and stale-response paths.

# Common Mistakes

- Treating connectivity as API reachability.
- Ignoring status codes and malformed JSON.
- Storing tokens in plain preferences.
- Retrying non-idempotent writes.
- No pagination, migration, or contract tests.

# Debugging

- Trace method/URI -> transport -> status -> body -> model -> state.
- Record sanitized timing and request ID.
- Distinguish network exception from HTTP status.
- Test with mocks and target devices.

# Interview Questions

1. Describe the request-response lifecycle.
2. Distinguish 401, 403, 404, and 500.
3. Explain timeout versus cancellation.
4. Explain REST versus GraphQL.
5. Explain safe POST retries.
6. Design pagination and infinite scroll.
7. Design WebSocket reconnect and event reconciliation.
8. Explain DTOs and JSON validation.

# Try It Yourself

1. Add a timeout to a request.
2. Implement typed status errors.
3. Create a mock-client test.
4. Trace a request with sanitized metadata.

# Challenge Project

Build a complete API-driven feature with typed models, repository, state management, pagination, cache, authentication, loading/error UI, and deterministic tests.

# Quick Revision Sheet

- Request: method + URI + headers + body.
- Response: status + headers + body.
- Validate status before decoding.
- Use typed models, repositories, and injected clients.
- Handle timeout, auth, pagination, cache, and stale results.
- Protect secrets; test realistic failures.

# Introduction

HTTP methods describe the intended action on a resource. Common methods are GET, POST, PUT, PATCH, and DELETE.

# Why this concept exists

Consistent methods make APIs easier to understand, cache, secure, and operate. They communicate whether a request reads data, creates it, replaces it, partially updates it, or deletes it.

# Real-world analogy

At a records desk, you can inspect a file, submit a new file, replace a file, edit one field, or remove a file. Each action has a distinct meaning.

# Internal Working

The client chooses a method and sends it to a resource URI. The server defines exact semantics, but common conventions are: GET is safe/read-oriented; PUT replaces or creates a resource at a known URI and is idempotent by HTTP semantics; PATCH applies partial changes; POST often creates or triggers processing; DELETE requests removal.

An API may have nonstandard behavior, so follow its contract. Idempotence matters for retries: repeating a request should not create unintended duplicate effects.

# Syntax

```dart
await http.get(uri);
await http.post(uri, body: jsonEncode(payload));
await http.put(uri, body: jsonEncode(payload));
await http.patch(uri, body: jsonEncode(changes));
await http.delete(uri);
```

# Complete Example 1

This runnable app lets users select a method and sends it to a public test endpoint. Add the `http` package. The endpoint is for demonstration and does not persist a real record.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const MethodsApp());

class MethodsApp extends StatelessWidget {
  const MethodsApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: MethodsPage());
}

class MethodsPage extends StatefulWidget {
  const MethodsPage({super.key});

  @override
  State<MethodsPage> createState() => _MethodsPageState();
}

class _MethodsPageState extends State<MethodsPage> {
  String _result = 'Choose a method';

  Future<void> _send(String method) async {
    final uri = Uri.https('httpbin.org', '/anything');
    try {
      final response = switch (method) {
        'GET' => await http.get(uri),
        'POST' => await http.post(uri, body: {'example': 'value'}),
        'PUT' => await http.put(uri, body: {'example': 'value'}),
        'PATCH' => await http.patch(uri, body: {'example': 'changed'}),
        _ => await http.delete(uri),
      };
      if (!mounted) return;
      setState(() => _result = '$method returned ${response.statusCode}');
    } catch (error) {
      if (!mounted) return;
      setState(() => _result = 'Request failed');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('HTTP methods')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_result),
              for (final method in ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'])
                TextButton(
                  onPressed: () => _send(method),
                  child: Text(method),
                ),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `_send` uses a shared test endpoint and chooses the HTTP client method with a Dart switch expression.
- GET sends no body; POST/PUT/PATCH send form-style example fields for this endpoint.
- DELETE makes a removal-style request.
- The result shows the method and returned status.
- Errors are handled as transport failures.
- A generated row of buttons triggers each method.

### Widget tree

```text
MaterialApp -> MethodsPage -> Scaffold -> Center -> Column
                                                 ├-> result
                                                 └-> method buttons
```

### Execution flow

```text
Select method -> request uses method-specific client call
             -> server routes action -> status response -> display
```

### ASCII UI preview

```text
+---------------------------+
|       HTTP methods        |
+---------------------------+
| Choose a method           |
| [ GET ]                   |
| [ POST ]                  |
| [ PUT ]                   |
| [ PATCH ]                 |
| [ DELETE ]                |
+---------------------------+
```

### Example explanation

The method is not just a label: it changes request semantics. This demo endpoint echoes requests and should not be treated as a resource API. Production payloads should be JSON or the specific content type expected by the server.

If a write method is retried blindly, it may duplicate work (especially POST). If the method contract is ignored, caches, proxies, and backend behavior may not match expectations.

**Common mistakes:** Do not use GET to perform a destructive or state-changing action.

**Professional use case:** Choose method semantics consistently for a versioned REST resource API.

# Complete Example 2

A JSON POST:

```dart
await http.post(
  uri,
  headers: {'content-type': 'application/json'},
  body: jsonEncode({'name': 'Ari'}),
);
```

The server creates or processes the resource according to the endpoint contract.

# Complete Example 3

A partial update:

```dart
await http.patch(
  Uri.https('api.example.com', '/v1/tasks/$id'),
  headers: {'content-type': 'application/json'},
  body: jsonEncode({'done': true}),
);
```

Only the supplied field is intended to change; API-specific patch format may differ.

# Advanced Example

Classify operations by safety and idempotence before retrying. GET is usually safe; PUT and DELETE are designed to be idempotent but server implementation matters; POST is commonly non-idempotent. Use idempotency keys for retryable creates.

# Under the Hood

The method is part of the HTTP request line and influences server routing and intermediary behavior. The body format is independent of method and is described by headers.

# Performance Notes

- Cache safe GET responses when permitted.
- Avoid unnecessary writes.
- Batch API operations if supported.
- Use idempotency to protect retries.

# Best Practices

- Follow resource semantics.
- Use the correct content type.
- Validate status codes.
- Document retry policy by method.
- Use API-specific PATCH format.

# Common Mistakes

- State changes through GET.
- Treating PUT and PATCH as identical.
- Retrying POST without deduplication.
- Assuming DELETE always returns a body.

# Debugging

- Verify method in request logs.
- Check server route and required body format.
- Compare response status against method contract.
- Test retries and duplicate submissions.

# Interview Questions

1. Which method is safe/read-oriented? GET.
2. How do PUT and PATCH differ? Full replacement versus partial modification by common convention.
3. What does idempotent mean? Repeating the same request has the same intended effect.
4. Why is POST retry riskier? It may create duplicate resources unless deduplicated.

# Try It Yourself

1. Define methods for a task resource.
2. Explain retry policy for each method.
3. Add JSON headers and body.
4. Compare PUT and PATCH server contracts.

# Challenge Project

Design a CRUD endpoint map for a notes API, including method, path, request body, response, status, and retry policy.

# Quick Revision Sheet

- GET read; POST create/process; PUT replace; PATCH partial update; DELETE remove.
- Follow the API contract.
- Idempotence informs retry safety.
- Headers describe body formats.

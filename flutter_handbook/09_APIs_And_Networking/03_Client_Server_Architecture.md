# Introduction

Client-server architecture separates an app that requests services (client) from a system that provides them (server). A Flutter app is usually a client; its backend stores shared data, enforces permissions, and exposes API endpoints.

# Why this concept exists

Central services allow multiple users and devices to share consistent information. They also provide a place for authorization, business rules, and data management that should not be trusted to a client.

# Real-world analogy

A customer places an order with a shop. The customer app can request and display an order, but the shop's system decides whether inventory exists and whether the customer is authorized.

# Internal Working

The Flutter client sends an HTTP request containing method, URI, headers, and optional body. The server authenticates the caller, validates input, applies business logic, reads/writes storage, and returns a response. The client checks status and payload, then updates presentation state.

The client is untrusted. Validation in Flutter improves UX but does not replace server-side validation or authorization. Never embed privileged server secrets in a mobile binary.

# Syntax

```dart
final response = await client.get(
  Uri.https('api.example.com', '/v1/profile'),
  headers: {'authorization': 'Bearer $token'},
);
```

# Complete Example 1

This runnable app displays a client-server flow using a public demo API. Add `http` to dependencies.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const ClientServerApp());

class ClientServerApp extends StatelessWidget {
  const ClientServerApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: ProfilePage());
}

class ProfilePage extends StatefulWidget {
  const ProfilePage({super.key});

  @override
  State<ProfilePage> createState() => _ProfilePageState();
}

class _ProfilePageState extends State<ProfilePage> {
  String? _name;
  String _state = 'Ready';

  Future<void> _load() async {
    setState(() => _state = 'Requesting server...');
    try {
      final uri = Uri.https('jsonplaceholder.typicode.com', '/users/1');
      final response = await http.get(uri);
      if (response.statusCode != 200) {
        throw Exception('HTTP ${response.statusCode}');
      }
      final json = jsonDecode(response.body) as Map<String, dynamic>;
      if (!mounted) return;
      setState(() {
        _name = json['name'] as String;
        _state = 'Profile loaded';
      });
    } catch (error) {
      if (!mounted) return;
      setState(() => _state = 'Could not load profile');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Client and server')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_state),
              if (_name != null) Text(_name!),
              FilledButton(onPressed: _load, child: const Text('Load profile')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- JSON decoding, widgets, and HTTP client are imported separately.
- `_load` changes UI to a request state, calls the server asynchronously, and checks HTTP status.
- The body is decoded into a map and the name extracted.
- `mounted` prevents updating after navigation/disposal.
- Errors produce a user-friendly state.
- The widget conditionally displays a name only after a valid response.

### Widget tree

```text
MaterialApp -> ProfilePage -> Scaffold -> Center -> Column
                                                 ├-> request state
                                                 ├-> optional name
                                                 └-> load button
```

### Execution flow

```text
Flutter client -> HTTPS request -> backend validation/business logic
Flutter client <- response status + JSON <- backend
decode -> state -> UI
```

### ASCII UI preview

```text
+---------------------------+
|   Client and server       |
+---------------------------+
| Profile loaded            |
| Leanne Graham              |
| [ Load profile ]           |
+---------------------------+
```

### Example explanation

The client requests a resource and renders a server-provided name. The demo backend is not authoritative for real user data. Production code would map a typed response model and enforce server-side authorization.

If status validation is removed, an error response can be misparsed. If the server trusts client-side checks alone, an attacker can bypass them using a different HTTP client.

**Common mistakes:** Never put database passwords or privileged API secrets in the Flutter app.

**Professional use case:** Flutter as a thin client for authenticated account and product services.

# Complete Example 2

A client-side validation improves usability:

```dart
if (email.trim().isEmpty) {
  showValidationMessage();
  return;
}
```

The server must repeat security-relevant validation because client code can be modified or bypassed.

# Complete Example 3

A server error should be translated into application state rather than rendered as raw HTML/JSON:

```dart
if (response.statusCode >= 500) {
  throw const ApiUnavailableException();
}
```

# Advanced Example

Separate UI, state management, repository, API client, and backend contracts. Use versioned endpoints, request IDs, idempotency where needed, and explicit authentication/authorization. For sensitive actions, validate permissions on every server request.

# Under the Hood

The app's HTTP client serializes messages; backend gateways and services route and process them; server databases store shared state. A client response is untrusted until validated and decoded. The UI updates only after state management receives the result.

# Performance Notes

- Cache appropriate read-only data.
- Avoid duplicate requests and N+1 API calls.
- Paginate large collections.
- Reuse HTTP connections.
- Keep server processing and mobile rendering responsive.

# Best Practices

- Enforce business rules and access control on the server.
- Validate and sanitize on both ends.
- Use HTTPS and secure token lifecycle.
- Version API contracts.
- Handle retries and duplicate submission safely.

# Common Mistakes

- Treating mobile validation as security.
- Embedding server secrets.
- Trusting client-supplied user IDs for authorization.
- Assuming network success means business success.
- Coupling UI directly to backend JSON.

# Debugging

- Trace client request and server logs with a request ID.
- Verify authorization and validation at server.
- Inspect sanitized request/response metadata.
- Separate client exceptions from backend status codes.

# Interview Questions

1. What is client-server architecture? A client requests services from a server that processes and returns results.
2. Why validate on the server? Clients are untrusted and can be bypassed.
3. Can an API secret be hidden in a Flutter binary? No reliable secrecy for distributed client code.
4. What is authorization? Determining whether an authenticated principal may perform an action.

# Try It Yourself

1. Add a request ID header.
2. Design server-side validation for a submitted form.
3. Map status codes to user-friendly errors.
4. Draw UI/repository/client/server/database layers.

# Challenge Project

Design an authenticated task service with client-side UX validation, server authorization, request tracing, and a retry-safe create operation.

# Quick Revision Sheet

- Flutter is a client; backend is authoritative for shared rules/data.
- Request -> server processing -> response -> model/state/UI.
- Validate on client for UX and server for security.
- Never embed privileged secrets.
- Separate UI, repository, API client, and backend.

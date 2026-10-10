# Introduction

PUT sends a representation to a known resource URI, commonly replacing the resource or creating it at that address. The API contract defines exact behavior.

# Why this concept exists

Clients need a clear operation for setting the complete state of a resource. PUT is conventionally idempotent, making repeated identical requests safer than repeated creates.

# Real-world analogy

PUT is replacing a complete form in a folder with a new complete version. Fields omitted from the representation may be reset or rejected, depending on the server contract.

# Internal Working

The client sends PUT to the resource's URI with a representation. The server authenticates, validates, and replaces or upserts the resource as documented. A successful response may return a representation, 204, or other status.

# Syntax

```dart
await http.put(
  Uri.https('api.example.com', '/v1/profile/42'),
  headers: {'content-type': 'application/json'},
  body: jsonEncode(profile.toJson()),
);
```

# Complete Example 1

This app sends a full demo profile representation to httpbin. Add `http`; the endpoint echoes requests and does not persist a profile.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const PutRequestApp());

class PutRequestApp extends StatelessWidget {
  const PutRequestApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: PutRequestPage());
}

class PutRequestPage extends StatefulWidget {
  const PutRequestPage({super.key});

  @override
  State<PutRequestPage> createState() => _PutRequestPageState();
}

class _PutRequestPageState extends State<PutRequestPage> {
  final _name = TextEditingController(text: 'Sam');
  String _status = 'Not saved';

  Future<void> _save() async {
    setState(() => _status = 'Saving...');
    try {
      final response = await http.put(
        Uri.https('httpbin.org', '/anything/profile/42'),
        headers: {'content-type': 'application/json'},
        body: jsonEncode({'id': 42, 'name': _name.text.trim()}),
      );
      if (!mounted) return;
      setState(() => _status = 'PUT returned ${response.statusCode}');
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Save failed');
    }
  }

  @override
  void dispose() {
    _name.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('PUT request')),
        body: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            children: [
              TextField(
                controller: _name,
                decoration: const InputDecoration(labelText: 'Full name'),
              ),
              FilledButton(onPressed: _save, child: const Text('Replace profile')),
              Text(_status),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `_name` holds editable profile data.
- `_save` displays progress and sends PUT to a URI naming a specific resource.
- The JSON body contains the complete demo representation.
- `Content-Type` declares JSON.
- The result status or transport failure is shown.
- The text controller is disposed.
- The UI consists of a labeled field, save action, and status.

### Widget tree

```text
MaterialApp → PutRequestPage → Scaffold → Column
                                           ├→ TextField
                                           ├→ Replace button
                                           └→ status
```

### Execution flow

```text
Edit full resource -> PUT /profile/42 -> server validates/replaces
                    <- status/result <- update UI
```

### ASCII UI preview

```text
+---------------------------+
|        PUT request        |
+---------------------------+
| Full name                 |
| [ Sam                  ]  |
| [   Replace profile    ]  |
| PUT returned 200          |
+---------------------------+
```

### Example explanation

PUT targets a known resource path and sends a representation. This demo echoes only. Production code must know whether omitted fields are replaced, defaulted, or rejected.

If the resource ID is missing, the request may target the wrong endpoint. If the full replacement contract is misunderstood, omitted fields can be lost. PUT is conventionally idempotent, but verify actual API behavior.

**Common mistakes:** Use PATCH when the API expects partial field updates; do not assume every server implements upsert semantics.

**Professional use case:** Save a complete editable profile or settings resource at a stable URI.

# Complete Example 2

PUT may be retried safely in many APIs when the same complete representation targets the same resource:

```dart
await client.put(resourceUri, body: encodedRepresentation);
```

The server must implement the expected idempotent semantics.

# Complete Example 3

Use optimistic concurrency with an ETag:

```dart
headers: {'if-match': etag}
```

The server can reject an update if the resource changed since it was read.

# Advanced Example

For concurrent edits, use server versions, ETags, or revision numbers. Surface conflicts and let users reconcile rather than silently overwriting another device's changes.

# Under the Hood

HTTP treats PUT as idempotent by semantics: repeating the same request should have the same intended state effect. Network delivery can still be ambiguous if the server commits but the response is lost.

# Performance Notes

- Send only required representation fields according to contract.
- Avoid resending large blobs for small changes.
- Use compression and concurrency control for large resources.

# Best Practices

- Use a stable resource URI.
- Understand full replacement semantics.
- Check status and response.
- Use concurrency validators for shared edits.
- Keep retries bounded and authenticated.

# Common Mistakes

- Confusing PUT with partial PATCH.
- Omitting required fields.
- Assuming a 200-only success.
- Blindly overwriting concurrent edits.

# Debugging

- Compare request body to full resource schema.
- Verify target URI and ID.
- Inspect ETag/version conflicts.
- Check server status and body.

# Interview Questions

1. What is PUT commonly used for? Replacing a resource at a known URI.
2. Is PUT idempotent? By HTTP semantics, intended to be.
3. How does it differ from PATCH? Full representation replacement versus partial modification by convention.
4. How handle concurrent updates? ETags/revisions and conflict UX.

# Try It Yourself

1. Add response parsing.
2. Simulate a conflict response.
3. Compare PUT and PATCH payloads.
4. Test repeating identical PUT.

# Challenge Project

Build a profile editor using PUT with ETag-based concurrency control and explicit conflict handling.

# Quick Revision Sheet

- PUT targets a known resource and typically replaces it.
- Use full representation per API contract.
- Repeated identical PUT should be idempotent by semantics.
- ETags/revisions help prevent lost updates.

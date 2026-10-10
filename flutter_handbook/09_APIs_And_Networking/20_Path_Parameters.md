# Introduction

A path parameter identifies a particular resource or nested resource in the URL path. For example, `/users/42` identifies user 42.

# Why this concept exists

Path parameters give resources stable, readable addresses. They distinguish one record from another, while query parameters more commonly filter or modify a collection request.

# Real-world analogy

A path parameter is a room number in a building address. Query parameters are extra directions about what information to bring from that room.

# Internal Working

The client builds a path from fixed segments and dynamic identifiers. IDs should be validated and encoded as URI path segments when they may contain reserved characters. The server routes the path, parses the identifier, and checks authorization.

# Syntax

```dart
final uri = Uri.https(
  'api.example.com',
  '/v1/users/${Uri.encodeComponent(userId)}',
);
```

# Complete Example 1

This app loads a user by ID from a demo endpoint. Add `http`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const PathParameterApp());

class PathParameterApp extends StatelessWidget {
  const PathParameterApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: PathParameterPage());
}

class PathParameterPage extends StatefulWidget {
  const PathParameterPage({super.key});

  @override
  State<PathParameterPage> createState() => _PathParameterPageState();
}

class _PathParameterPageState extends State<PathParameterPage> {
  final _id = TextEditingController(text: '1');
  String _result = 'Enter an ID';

  Future<void> _load() async {
    final id = _id.text.trim();
    if (id.isEmpty) {
      setState(() => _result = 'ID is required');
      return;
    }
    try {
      final uri = Uri.https(
        'jsonplaceholder.typicode.com',
        '/users/${Uri.encodeComponent(id)}',
      );
      final response = await http.get(uri);
      if (response.statusCode != 200) {
        throw Exception('HTTP ${response.statusCode}');
      }
      final user = jsonDecode(response.body) as Map<String, dynamic>;
      if (!mounted) return;
      setState(() => _result = user['name'] as String);
    } catch (error) {
      if (!mounted) return;
      setState(() => _result = 'Could not load user');
    }
  }

  @override
  void dispose() {
    _id.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Path parameter')),
        body: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            children: [
              TextField(
                controller: _id,
                decoration: const InputDecoration(labelText: 'User ID'),
              ),
              FilledButton(onPressed: _load, child: const Text('Load user')),
              Text(_result),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `_id` collects the identifier; `_result` stores the visible response.
- `_load` trims and validates the ID.
- `Uri.encodeComponent` escapes the dynamic path value.
- The GET requests a specific `/users/{id}` resource.
- Status is checked before decoding the response map.
- The user name is extracted and displayed.
- The text controller is disposed.

### Widget tree

```text
MaterialApp → PathParameterPage → Scaffold → Padding → Column
                                                    ├→ ID field
                                                    ├→ Load button
                                                    └→ result
```

### Execution flow

```text
ID input -> validate/encode -> GET /users/{id}
          -> server routes specific record -> JSON -> displayed name
```

### ASCII UI preview

```text
+---------------------------+
|      Path parameter       |
+---------------------------+
| User ID                   |
| [ 1                    ]  |
| [      Load user       ]  |
| Leanne Graham              |
+---------------------------+
```

### Example explanation

Path parameters identify the resource. For numeric IDs, validate numeric form too; this sample uses encoding so arbitrary input cannot alter path structure. Authorization must still happen on the server.

If the dynamic ID is interpolated without encoding, reserved characters may change path interpretation. If client validation is treated as authorization, attackers can request other IDs directly.

**Common mistakes:** A hidden button or disabled UI is not access control; server must authorize each ID.

# Complete Example 2

Nested resource path:

```dart
Uri.https('api.example.com', '/users/$userId/orders/$orderId');
```

Both identifiers need encoding/validation, and the server must verify that the order belongs to that user.

# Complete Example 3

A resource identifier is not normally a query filter:

```text
GET /products/sku-123  # one known item
GET /products?category=books  # collection filtered by category
```

# Advanced Example

Use opaque identifiers safely and avoid exposing sensitive internals in URLs. API gateways log paths, so do not place secrets in path segments. Decide canonicalization and case sensitivity for IDs.

# Under the Hood

URI path segments are encoded and sent as part of the request target. Server routing matches segments and validates the identifier before data access. Proxies and logs may record the full path.

# Performance Notes

- Cache item requests by full resource identity.
- Avoid fetching one detail request per list item when an API can batch.
- Validate IDs before sending.

# Best Practices

- Use path for resource identity and query for collection options.
- Encode dynamic segments.
- Use stable IDs.
- Enforce server-side ownership/authorization.
- Avoid sensitive values in URLs.

# Common Mistakes

- Unescaped path interpolation.
- Trusting user ID from client without authorization.
- Confusing path ID with query filter.
- Logging private identifiers unnecessarily.

# Debugging

- Inspect final path and encoding.
- Check 404 versus 403 behavior.
- Verify nested ownership at server.
- Test IDs containing reserved characters where supported.

# Interview Questions

1. What is a path parameter? A dynamic URI segment identifying a resource.
2. How does it differ from query parameter? Path identifies resource; query commonly filters/configures.
3. Why encode IDs? Reserved characters may alter URI structure.
4. Is client-side ID validation security? No; server authorization is required.

# Try It Yourself

1. Add numeric validation.
2. Build nested order path.
3. Compare 404 and 403 responses.
4. Cache details by full URI.

# Challenge Project

Build a user detail route that fetches by path ID, handles invalid/missing IDs, and ensures account ownership is validated server-side.

# Quick Revision Sheet

- Path segments identify resources.
- Encode/validate dynamic segments.
- Query parameters filter/configure.
- Server validates resource access.
- Never put credentials in a URL.

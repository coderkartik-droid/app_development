# Introduction

Encoding converts Dart data into JSON text for a request. Decoding converts JSON text from a response into dynamic Dart structures. Model mapping then turns those structures into application types.

# Why this concept exists

HTTP transports bytes, not Dart objects. Encoding and decoding define how each side exchanges structured data.

# Real-world analogy

Encoding is packing objects into a standard shipping box with a manifest. Decoding is opening the box and checking that each labeled item is present and the right type.

# Internal Working

`jsonEncode` serializes supported Dart primitives, lists, and maps to a string. Custom objects need a `toJson` representation or a supported converter. `jsonDecode` parses JSON into dynamic values. Both can throw on unsupported or invalid data.

# Syntax

```dart
final body = jsonEncode({'title': 'Hello', 'done': false});
final decoded = jsonDecode(responseBody) as Map<String, dynamic>;
```

# Complete Example 1

This runnable app encodes a Dart map, decodes it again, and displays the result.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';

void main() => runApp(const EncodeDecodeApp());

class EncodeDecodeApp extends StatelessWidget {
  const EncodeDecodeApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: EncodeDecodePage());
}

class EncodeDecodePage extends StatelessWidget {
  const EncodeDecodePage({super.key});

  @override
  Widget build(BuildContext context) {
    final original = <String, Object>{
      'title': 'Pack lunch',
      'done': false,
      'priority': 2,
    };
    final encoded = jsonEncode(original);
    final decoded = jsonDecode(encoded) as Map<String, dynamic>;
    return Scaffold(
      appBar: AppBar(title: const Text('Encode and decode')),
      body: Padding(
        padding: const EdgeInsets.all(20),
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Encoded JSON'),
            Text(encoded),
            const SizedBox(height: 16),
            Text('Title: ${decoded['title']}'),
            Text('Done: ${decoded['done']}'),
          ],
        ),
      ),
    );
  }
}
```

### Line-by-line explanation

- `dart:convert` provides encoding and decoding.
- `original` is a map of JSON-compatible values.
- `jsonEncode` turns the map into a JSON string suitable for an HTTP body.
- `jsonDecode` parses that string back into a dynamic map.
- The UI shows wire text and selected decoded fields.

### Widget tree

```text
MaterialApp -> EncodeDecodePage -> Scaffold -> Padding -> Column
                                                     ├-> encoded JSON
                                                     ├-> title
                                                     └-> done
```

### Execution flow

```text
Dart map -> jsonEncode -> JSON text -> transport
JSON response text -> jsonDecode -> dynamic structure -> model mapping
```

### ASCII UI preview

```text
+---------------------------+
|   Encode and decode       |
+---------------------------+
| {"title":"Pack lunch"...} |
| Title: Pack lunch         |
| Done: false               |
+---------------------------+
```

### Example explanation

The round trip works because the map contains JSON-supported values. `jsonEncode` does not know how to serialize arbitrary class instances unless they are converted. `jsonDecode` returns dynamic data, so the app should validate and map it.

If the `dart:convert` import is removed, these functions are unavailable. If a map contains an unsupported object, encoding throws. If response text is malformed, decoding throws.

**Common mistakes:** Do not assume JSON numeric values always decode to the same Dart numeric subtype.

**Professional use case:** Encode request DTOs and decode response DTOs at the API boundary.

# Complete Example 2

Encode a model via `toJson`:

```dart
final body = jsonEncode(task.toJson());
```

The model method controls wire field names and types. Keep secrets and local-only fields out of the representation.

# Complete Example 3

Decode an array:

```dart
final rows = jsonDecode(body) as List<dynamic>;
final maps = rows.cast<Map<String, dynamic>>();
```

Validate each row before mapping; `cast` creates a typed view and may throw when an element does not match.

# Advanced Example

Use explicit wire DTOs when API shape differs from domain behavior. Handle dates, enums, large integers, and polymorphic data with documented converters. Test round-trip behavior and compatibility with server fixtures.

# Under the Hood

Encoding recursively converts JSON-compatible values into text. Decoding parses text into generic Dart values. Typed model construction is a separate step, which is where required fields and defaults should be enforced.

# Performance Notes

- Avoid encoding large objects repeatedly.
- Use streaming or isolate computation for large payloads when warranted.
- Request smaller payloads.
- Reuse parsed models rather than decoding the same body multiple times.

# Best Practices

- Keep encoding at API boundaries.
- Use `toJson`/`fromJson` or code generation.
- Validate dynamic values.
- Handle malformed payloads as parsing errors.
- Exclude credentials and irrelevant fields.

# Common Mistakes

- Passing a class instance directly to `jsonEncode`.
- Treating decoded dynamic as trustworthy.
- Forgetting request `Content-Type`.
- Silently defaulting invalid required fields.
- Logging entire private payloads.

# Debugging

- Print a sanitized encoded sample.
- Verify key names and nesting.
- Inspect the exact parser path causing a type error.
- Compare against API contract and fixture.
- Distinguish invalid JSON syntax from valid JSON with wrong shape.

# Interview Questions

1. What does `jsonEncode` accept? JSON-compatible Dart values or values with conversion support.
2. What does `jsonDecode` produce? Dynamic structures.
3. Why separate DTO and domain model? Wire schemas and app concepts have different responsibilities.
4. What happens on invalid JSON? A format/parsing exception is thrown.

# Try It Yourself

1. Encode a nested task object.
2. Add a date converter.
3. Test missing required and optional fields.
4. Verify that private fields do not appear in JSON.

# Challenge Project

Create request/response DTOs for a profile API, with JSON round-trip tests, date parsing, enum fallback, and clear errors for malformed required fields.

# Quick Revision Sheet

- `jsonEncode`: Dart JSON-compatible value → text.
- `jsonDecode`: text → dynamic Dart structures.
- Model mapping is a separate validation step.
- Set content type for JSON requests.
- Test edge cases and keep secrets out of payloads.

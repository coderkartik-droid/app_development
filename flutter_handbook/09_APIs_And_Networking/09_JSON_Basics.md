# Introduction

JSON (JavaScript Object Notation) is a text format for representing values such as objects, arrays, strings, numbers, booleans, and null. Web APIs commonly use it for request and response bodies.

# Why this concept exists

Clients and servers may use different programming languages. JSON provides a compact, language-neutral representation that both sides can encode and decode.

# Real-world analogy

JSON is a standard form with labeled fields. Different people can fill out or read it as long as they agree on field names and value types.

# Internal Working

The HTTP body arrives as bytes, commonly decoded as UTF-8 text. Dart's `jsonDecode` parses that text into dynamic values: `Map<String, dynamic>`, `List<dynamic>`, strings, numbers, booleans, or null. The application validates shape and maps it to typed models.

# Syntax

```json
{
  "id": 5,
  "title": "Learn JSON",
  "completed": false
}
```

# Complete Example 1

This runnable Flutter screen decodes a local JSON string and presents its fields. It needs no networking package.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';

const sampleJson = '''
{
  "name": "Mina",
  "favoriteTopics": ["Flutter", "APIs"],
  "active": true
}
''';

void main() => runApp(const JsonBasicsApp());

class JsonBasicsApp extends StatelessWidget {
  const JsonBasicsApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: JsonBasicsPage());
}

class JsonBasicsPage extends StatelessWidget {
  const JsonBasicsPage({super.key});

  @override
  Widget build(BuildContext context) {
    final data = jsonDecode(sampleJson) as Map<String, dynamic>;
    final topics = data['favoriteTopics'] as List<dynamic>;
    return Scaffold(
      appBar: AppBar(title: const Text('JSON basics')),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(data['name'] as String),
            Text('Active: ${data['active']}'),
            Text('Topics: ${topics.join(', ')}'),
          ],
        ),
      ),
    );
  }
}
```

### Line-by-line explanation

- `dart:convert` includes `jsonDecode`.
- `sampleJson` is a multiline JSON object stored as a Dart string.
- `jsonDecode` returns `dynamic`, so the code casts the top-level object to a map.
- The topics field is an array, represented as a Dart list.
- `Text` widgets display the string, boolean, and joined list.

### Widget tree

```text
MaterialApp -> JsonBasicsPage -> Scaffold -> Center -> Column
                                                    ├-> name
                                                    ├-> active flag
                                                    └-> topic list
```

### Execution flow

```text
JSON text -> jsonDecode -> dynamic map/list -> typed access -> widgets
```

### ASCII UI preview

```text
+---------------------------+
|        JSON basics        |
+---------------------------+
| Mina                      |
| Active: true              |
| Topics: Flutter, APIs     |
+---------------------------+
```

### Example explanation

JSON objects map naturally to Dart maps, and arrays map to lists. Casting assumes the sample is valid. Real API responses are untrusted and need validation with meaningful parsing errors.

If the JSON has invalid syntax, `jsonDecode` throws. If `favoriteTopics` is absent or not a list, the cast fails. A typed model centralizes these checks.

**Common mistakes:** JSON has `null`, not an undefined value; numbers may decode as `int` or `double`.

**Professional use case:** Transfer compact structured records between Flutter and web services.

# Complete Example 2

JSON array:

```json
[{"id": 1, "name": "A"}, {"id": 2, "name": "B"}]
```

Decode the top-level value as a list and validate each element as an object.

# Complete Example 3

JSON null and booleans:

```json
{"nickname": null, "enabled": false}
```

Use nullable Dart properties for nullable fields; do not treat `false` as missing.

# Advanced Example

Define schema expectations, tolerate optional fields, reject invalid required fields, and consider unknown extra fields forward-compatible. Avoid assuming exact key ordering; JSON object order is not a semantic contract.

# Under the Hood

The JSON parser tokenizes text and creates Dart dynamic structures. Model mapping is separate and should convert from flexible wire representation to typed domain objects.

# Performance Notes

- Avoid parsing huge payloads on the UI isolate.
- Paginate large collections and reduce payload size.
- Use generated serializers for large models when beneficial.
- Do not repeatedly decode the same response.

# Best Practices

- Validate shape and types.
- Use typed models.
- Handle nulls and optional fields intentionally.
- Keep wire naming separate from UI naming when needed.
- Avoid logging full private JSON bodies.

# Common Mistakes

- Assuming `jsonDecode` returns a typed model.
- Force-casting all fields.
- Confusing absent, null, false, and empty values.
- Depending on key order.
- Parsing error pages as JSON without checking content.

# Debugging

- Inspect content type and sanitized body sample.
- Check the top-level shape (object vs list).
- Report the failing key/path in parser errors.
- Validate server contract with fixtures.

# Interview Questions

1. What types can JSON represent? Objects, arrays, strings, numbers, booleans, and null.
2. What does `jsonDecode` return? Dynamic Dart structures, not an application model automatically.
3. Is object key order significant? No, not for normal JSON semantics.
4. How should optional fields be handled? Nullable/defaulted according to the contract.

# Try It Yourself

1. Decode a nested object and array.
2. Handle a missing optional field.
3. Test invalid JSON syntax.
4. Write a typed model for the sample.

# Challenge Project

Build a JSON fixture parser for a nested catalog response, with helpful errors for missing IDs, invalid types, and optional metadata.

# Quick Revision Sheet

- JSON is a text interchange format.
- `jsonDecode` returns dynamic maps/lists/primitives.
- Validate before trusting fields.
- Distinguish null, absent, false, and empty.
- Map wire data into typed models.

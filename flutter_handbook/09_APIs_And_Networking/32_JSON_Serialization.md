# Introduction

JSON serialization converts between Dart objects and JSON-compatible values. Deserialization reconstructs Dart objects from decoded JSON structures.

# Why this concept exists

APIs exchange language-neutral data. Serialization gives Flutter code a repeatable boundary between typed objects and wire representations.

# Real-world analogy

Serialization is packing a typed object into a standard shipping manifest; deserialization reads the manifest and reconstructs a typed parcel.

# Internal Working

`jsonEncode` serializes maps/lists/primitives; `jsonDecode` parses JSON text into dynamic Dart structures. Custom classes need explicit `toJson`/`fromJson` functions or generated serializers. Generation reduces repetitive code but adds a build step and still requires schema policy.

# Syntax

```dart
final text = jsonEncode(product.toJson());
final product = Product.fromJson(jsonDecode(text) as Map<String, dynamic>);
```

# Complete Example 1

This runnable app serializes and deserializes a typed task using manual methods.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';

class TaskDto {
  const TaskDto({required this.id, required this.title, required this.done});
  final int id;
  final String title;
  final bool done;

  Map<String, Object> toJson() => {
        'id': id,
        'title': title,
        'done': done,
      };

  factory TaskDto.fromJson(Map<String, dynamic> json) => TaskDto(
        id: json['id'] as int,
        title: json['title'] as String,
        done: json['done'] as bool,
      );
}

void main() => runApp(const SerializationApp());

class SerializationApp extends StatelessWidget {
  const SerializationApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: SerializationPage());
}

class SerializationPage extends StatelessWidget {
  const SerializationPage({super.key});

  @override
  Widget build(BuildContext context) {
    const task = TaskDto(id: 8, title: 'Review API', done: false);
    final encoded = jsonEncode(task.toJson());
    final decodedMap = jsonDecode(encoded) as Map<String, dynamic>;
    final restored = TaskDto.fromJson(decodedMap);
    return Scaffold(
      appBar: AppBar(title: const Text('JSON serialization')),
      body: Center(
        child: Text('${restored.title} (done: ${restored.done})'),
      ),
    );
  }
}
```

### Line-by-line explanation

- `TaskDto` is immutable data representing the JSON contract.
- `toJson` returns JSON-supported values in a map.
- `fromJson` validates and constructs a typed DTO.
- `jsonEncode` turns the map into JSON text.
- `jsonDecode` parses the text back to a dynamic map.
- The UI displays the reconstructed model, demonstrating a round trip.

### Widget tree

```text
MaterialApp → SerializationPage → Scaffold → Center → Text(restored model)
```

### Execution flow

```text
TaskDto -> toJson -> jsonEncode -> text
text -> jsonDecode -> fromJson -> TaskDto -> UI
```

### ASCII UI preview

```text
+---------------------------+
|   JSON serialization      |
+---------------------------+
| Review API (done: false)  |
+---------------------------+
```

### Example explanation

The round trip proves the mapping for this data shape. A production model should account for optional fields, server naming, dates, and unknown enum values.

If `toJson` is omitted, `jsonEncode(task)` cannot automatically serialize this class. If casts are removed without validation, invalid dynamic values can propagate deeper into the app.

**Common mistakes:** Do not serialize local-only fields or secrets just because they exist on a model.

# Complete Example 2

A generated serializer commonly emits methods equivalent to:

```dart
Map<String, dynamic> _$TaskToJson(Task value) => {'id': value.id};
```

Use project generator tooling and keep generated code aligned with source annotations.

# Complete Example 3

Date values need a wire convention:

```dart
'createdAt': createdAt.toUtc().toIso8601String()
```

Parse back with `DateTime.parse` and normalize timezone expectations.

# Advanced Example

For very large payloads, consider generated parsing, isolates, or streaming JSON. Avoid unsafe runtime reflection; code generation is generally more predictable in Flutter release builds.

# Under the Hood

The serializer recursively walks JSON-compatible maps/lists and produces UTF-8-compatible text. Deserialization creates generic structures, then model code applies type checks and converters.

# Performance Notes

- Use generated serialization for broad repetitive models.
- Avoid encoding unchanged objects.
- Keep payloads narrow.
- Profile large JSON parsing on lower-end hardware.

# Best Practices

- Keep serialization at transport boundaries.
- Define explicit JSON field names.
- Validate and test round trips.
- Use UTC timestamps and clear null semantics.
- Regenerate code after schema changes.

# Common Mistakes

- Passing custom objects directly to `jsonEncode`.
- Failing to regenerate output.
- Losing timezone information.
- Silently defaulting required data.
- Including secrets in `toJson`.

# Debugging

- Compare encoded JSON with API contract.
- Check generator output/errors.
- Add fixtures for optional and malformed fields.
- Report JSON key paths in parse errors.

# Interview Questions

1. What is serialization? Converting objects to a transferable/storable representation.
2. What does `jsonDecode` return? Dynamic generic structures.
3. Why generated serialization? Reduce boilerplate and provide consistent mapping.
4. Does `toJson` need security review? Yes; it controls which model fields are exposed.

# Try It Yourself

1. Add a nullable field to round trip.
2. Add UTC date conversion.
3. Test omitted and null fields.
4. Generate serializers for a nested response.

# Challenge Project

Build a serialization layer for nested order and line-item DTOs with timestamps, optional values, and round-trip fixture tests.

# Quick Revision Sheet

- `toJson` maps model to map; `fromJson` reconstructs.
- `jsonEncode/jsonDecode` operate on supported generic values.
- Use converters for dates/enums.
- Generated code must be regenerated.
- Exclude secrets and test edge cases.

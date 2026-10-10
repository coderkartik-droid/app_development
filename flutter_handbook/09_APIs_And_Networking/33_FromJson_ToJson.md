# Introduction

`fromJson` and `toJson` are conventional model methods. `fromJson` builds a Dart object from decoded JSON; `toJson` creates a map suitable for encoding into a request.

# Why this concept exists

Keeping conversion in one place prevents every screen and service from manually reading or constructing JSON keys.

# Real-world analogy

`fromJson` is unpacking a labeled kit into a usable tool; `toJson` is packing the tool back into the standard kit format.

# Internal Working

`jsonDecode` returns dynamic values. `fromJson` should validate the top-level map and field types, handle optional fields, and convert wire-specific forms. `toJson` should include only fields the API expects and use its exact naming convention.

# Syntax

```dart
factory User.fromJson(Map<String, dynamic> json) => ...;
Map<String, Object?> toJson() => ...;
```

# Complete Example 1

This runnable app demonstrates a task model round trip and displays the result.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';

class UserProfile {
  const UserProfile({
    required this.id,
    required this.displayName,
    this.avatarUrl,
  });

  final String id;
  final String displayName;
  final String? avatarUrl;

  factory UserProfile.fromJson(Map<String, dynamic> json) {
    return UserProfile(
      id: json['id'] as String,
      displayName: json['display_name'] as String,
      avatarUrl: json['avatar_url'] as String?,
    );
  }

  Map<String, Object?> toJson() => {
        'id': id,
        'display_name': displayName,
        'avatar_url': avatarUrl,
      };
}

void main() => runApp(const FromToJsonApp());

class FromToJsonApp extends StatelessWidget {
  const FromToJsonApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: FromToJsonPage());
}

class FromToJsonPage extends StatelessWidget {
  const FromToJsonPage({super.key});

  @override
  Widget build(BuildContext context) {
    const sample = '{"id":"u-9","display_name":"Kai","avatar_url":null}';
    final decoded = jsonDecode(sample) as Map<String, dynamic>;
    final user = UserProfile.fromJson(decoded);
    final encodedAgain = jsonEncode(user.toJson());
    return Scaffold(
      appBar: AppBar(title: const Text('fromJson / toJson')),
      body: Center(
        child: Text('${user.displayName}\n$encodedAgain', textAlign: TextAlign.center),
      ),
    );
  }
}
```

### Line-by-line explanation

- `UserProfile` holds a required ID/name and nullable avatar URL.
- `fromJson` maps snake_case wire keys to Dart camelCase properties.
- Required fields use type checks; avatar URL allows null.
- `toJson` maps Dart fields back to the server's expected key names.
- The page decodes a sample, constructs a model, encodes it again, and displays both identity and JSON.

### Widget tree

```text
MaterialApp → FromToJsonPage → Scaffold → Center → Text
```

### Execution flow

```text
JSON string -> jsonDecode -> fromJson -> typed object
typed object -> toJson -> jsonEncode -> JSON string
```

### ASCII UI preview

```text
+---------------------------+
|   fromJson / toJson       |
+---------------------------+
| Kai                       |
| {"id":"u-9",...}          |
+---------------------------+
```

### Example explanation

Conversion functions document the API's field naming and nullability. The sample's nullable cast accepts a null avatar but will throw if a non-string value appears.

If `toJson` uses `displayName` instead of `display_name`, the backend may ignore the field. If required values are made nullable without reason, invalid responses can silently pass.

**Common mistakes:** Keep server naming in serialization methods, not throughout Dart identifiers.

# Complete Example 2

Parse optional integer fields robustly:

```dart
final count = (json['count'] as num?)?.toInt() ?? 0;
```

This accepts an integer or double representation, but use it only if the API contract permits both.

# Complete Example 3

Map a list:

```dart
final users = (json['users'] as List<dynamic>)
    .map((item) => UserProfile.fromJson(item as Map<String, dynamic>))
    .toList();
```

Validate the envelope and each element; malformed rows should have an explicit policy.

# Advanced Example

Avoid permissive conversions that hide breaking API changes. In a production parser, include field paths in errors, support unknown enum values intentionally, and separate API DTOs from domain rules.

# Under the Hood

Factories execute at runtime over dynamic maps. They enforce a boundary between flexible external input and statically typed application code. `toJson` controls outbound data exposure.

# Performance Notes

- Generate repetitive conversion code.
- Avoid multiple round trips through JSON text when a map is already available.
- Parse large lists in a background isolate if profiling shows jank.

# Best Practices

- Keep conversions in model/DTO classes.
- Match exact server field names.
- Define null and default semantics.
- Test both directions.
- Ensure outbound maps exclude secrets.

# Common Mistakes

- Wrong JSON key casing.
- Unchecked casts on untrusted data.
- Treating null as absent without contract.
- Serializing internal fields accidentally.

# Debugging

- Inspect a sanitized JSON fixture.
- Log parse path/key, not full sensitive body.
- Compare `toJson` output against API schema.
- Test null, missing, wrong type, and unknown values.

# Interview Questions

1. What does a factory `fromJson` do? Constructs a typed object from a JSON map.
2. Why use different Dart/wire names? Keep idiomatic Dart names while matching backend schema.
3. What should `toJson` exclude? Secrets and fields not accepted by the API.
4. Why test round trips? Ensure mapping is consistent.

# Try It Yourself

1. Add `created_at` as a parsed UTC DateTime.
2. Add unknown enum fallback.
3. Write round-trip tests.
4. Test wrong type for a required field.

# Challenge Project

Create a profile DTO with nested address and settings, snake_case mapping, optional fields, date parsing, and defensive tests.

# Quick Revision Sheet

- `fromJson`: wire map to typed object.
- `toJson`: typed object to API map.
- Match exact keys; validate types.
- Define nullable/default behavior.
- Test round trips and secret exclusion.

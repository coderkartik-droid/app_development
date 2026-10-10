# Introduction

An API response model is a Dart type that represents data returned by a server. Instead of passing dynamic JSON maps throughout the UI, a model gives fields clear names and types.

# Why this concept exists

Raw JSON is flexible but unsafe to use everywhere. A model catches mapping problems near the API boundary and makes UI code easier to read, test, and refactor.

# Real-world analogy

A response model is a labeled form that turns a delivery box into organized fields: tracking number, destination, and delivery status.

# Internal Working

The HTTP client returns bytes or text. A JSON decoder turns text into maps/lists, then a factory or generated serializer maps those dynamic values into typed Dart objects. The model may be a wire DTO; business logic can map it to a separate domain model.

# Syntax

```dart
factory Product.fromJson(Map<String, dynamic> json) => Product(
  id: json['id'] as int,
  name: json['name'] as String,
);
```

# Complete Example 1

This runnable app fetches one demo post, maps it to a typed response model, and displays it. Add `http`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

class PostResponse {
  const PostResponse({
    required this.id,
    required this.title,
    required this.body,
  });

  final int id;
  final String title;
  final String body;

  factory PostResponse.fromJson(Map<String, dynamic> json) {
    return PostResponse(
      id: json['id'] as int,
      title: json['title'] as String,
      body: json['body'] as String,
    );
  }
}

void main() => runApp(const ResponseModelApp());

class ResponseModelApp extends StatelessWidget {
  const ResponseModelApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: ResponseModelPage());
}

class ResponseModelPage extends StatefulWidget {
  const ResponseModelPage({super.key});

  @override
  State<ResponseModelPage> createState() => _ResponseModelPageState();
}

class _ResponseModelPageState extends State<ResponseModelPage> {
  late Future<PostResponse> _post;

  @override
  void initState() {
    super.initState();
    _post = _fetchPost();
  }

  Future<PostResponse> _fetchPost() async {
    final response = await http.get(
      Uri.https('jsonplaceholder.typicode.com', '/posts/1'),
    );
    if (response.statusCode != 200) {
      throw Exception('HTTP ${response.statusCode}');
    }
    final json = jsonDecode(response.body) as Map<String, dynamic>;
    return PostResponse.fromJson(json);
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Response model')),
        body: FutureBuilder<PostResponse>(
          future: _post,
          builder: (context, snapshot) {
            if (snapshot.hasError) return const Center(child: Text('Load failed'));
            if (!snapshot.hasData) {
              return const Center(child: CircularProgressIndicator());
            }
            final post = snapshot.data!;
            return Padding(
              padding: const EdgeInsets.all(24),
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Text('#${post.id} ${post.title}'),
                  const SizedBox(height: 12),
                  Text(post.body),
                ],
              ),
            );
          },
        ),
      );
}
```

### Line-by-line explanation

- `PostResponse` holds the typed ID, title, and body from the server.
- `fromJson` maps each JSON key to the corresponding field and checks expected Dart types.
- `initState` creates the request future once.
- `_fetchPost` performs GET, validates status, decodes the JSON object, and delegates mapping to the factory.
- `FutureBuilder<PostResponse>` ensures the builder receives a typed model.
- The UI displays the model fields and handles loading/error.

### Widget tree

```text
MaterialApp → ResponseModelPage → Scaffold → FutureBuilder<PostResponse>
                                           ├→ progress
                                           ├→ error
                                           └→ title/body
```

### Execution flow

```text
HTTP text → JSON map → PostResponse.fromJson → typed future → widgets
```

### ASCII UI preview

```text
+---------------------------+
|     Response model        |
+---------------------------+
| #1 sunt aut facere...     |
|                           |
| quia et suscipit...       |
+---------------------------+
```

### Example explanation

The model isolates JSON keys from the widget. If the API renames a key, the mapping fails at the boundary rather than spreading dynamic lookups throughout the app. This basic cast-based mapper throws when a required field is absent or has the wrong type.

If the model is removed, UI code must repeatedly access `dynamic` maps. If response status checking is removed, an error payload may be mapped as a valid post.

**Common mistakes:** Avoid using broad `dynamic` fields throughout the app; use nullable fields only when the API contract allows absence or null.

**Professional use case:** Strongly typed API DTOs make response handling, testing, and schema changes safer.

# Complete Example 2

A nullable field can be modeled deliberately:

```dart
final String? subtitle;
```

Map absent and explicit null values according to the backend contract; do not make required data nullable merely to silence parsing errors.

# Complete Example 3

Map a response envelope:

```dart
class ApiEnvelope<T> {
  const ApiEnvelope({required this.data, this.nextCursor});
  final T data;
  final String? nextCursor;
}
```

An envelope can carry pagination metadata, but parsing its generic payload requires a typed converter or explicit mapper.

# Advanced Example

Separate wire DTOs from domain models when API naming, nullability, or structure differs from the app's business concepts. Add validation for numeric ranges, date parsing, unknown enum values, and server schema evolution.

# Under the Hood

JSON decoding creates generic Dart values. The factory performs runtime checks and constructs an immutable object. Generated serializers can reduce boilerplate but still need defaults, converters, and tests.

# Performance Notes

- Generate serializers for large model sets when useful.
- Avoid repeated decoding/mapping of the same response.
- Paginate lists rather than mapping huge payloads.
- Move heavy parsing off the UI isolate if profiling warrants it.

# Best Practices

- Keep models immutable.
- Map at the API boundary.
- Validate required fields and document nullability.
- Use stable enum/date converters.
- Test representative and malformed payload fixtures.

# Common Mistakes

- Confusing decoded maps with typed models.
- Ignoring null and absent-field behavior.
- Making every field dynamic.
- Parsing dates or enums ad hoc in widgets.
- Assuming code generation replaces validation.

# Debugging

- Log the failing JSON path, not private payloads.
- Compare fixture keys/types to API schema.
- Test missing, null, wrong-type, and unknown values.
- Check envelope vs top-level list shape.

# Interview Questions

1. Why create response models? Type safety and isolation of wire mapping.
2. What is a DTO? A data transfer object shaped for communication across a boundary.
3. When separate DTO and domain model? When wire representation differs from application semantics.
4. Does generated JSON code validate business rules? Not automatically.

# Try It Yourself

1. Add an optional author field.
2. Parse a list of response models.
3. Add an enum converter with unknown fallback.
4. Write tests for malformed types.

# Challenge Project

Create typed models for a paginated catalog API with optional images, enum status, timestamp parsing, and descriptive parser errors.

# Quick Revision Sheet

- Decode JSON, then map it to typed models.
- Keep mapping out of widgets.
- Validate required fields and nullability.
- Separate wire DTO/domain models when useful.
- Test valid, missing, null, and invalid payloads.

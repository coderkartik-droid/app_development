# Introduction

A GET request asks a server to return a representation of a resource. It is the most common first API call because it reads data without intentionally changing server state.

# Why this concept exists

Mobile screens need data from remote services. GET provides a standard request shape for retrieving it and usually allows safe caching.

# Real-world analogy

GET is asking a librarian to show a book's catalogue record. The request should not change the catalogue.

# Internal Working

The client constructs a URI, sends GET, awaits response bytes, checks the status, decodes the representation, maps it to a model, and updates UI state. GET should generally be safe and idempotent; API contracts can have unusual behavior, but state changes through GET are discouraged.

# Syntax

```dart
final response = await http.get(Uri.https('api.example.com', '/items/1'));
```

# Complete Example 1

This runnable screen loads a single public post. Add `http`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const GetRequestApp());

class GetRequestApp extends StatelessWidget {
  const GetRequestApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: GetPostPage());
}

class GetPostPage extends StatefulWidget {
  const GetPostPage({super.key});

  @override
  State<GetPostPage> createState() => _GetPostPageState();
}

class _GetPostPageState extends State<GetPostPage> {
  late Future<String> _title;

  @override
  void initState() {
    super.initState();
    _title = _load();
  }

  Future<String> _load() async {
    final response = await http.get(
      Uri.https('jsonplaceholder.typicode.com', '/posts/1'),
    );
    if (response.statusCode != 200) {
      throw Exception('Request returned ${response.statusCode}');
    }
    final data = jsonDecode(response.body) as Map<String, dynamic>;
    return data['title'] as String;
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('First GET request')),
        body: Center(
          child: FutureBuilder<String>(
            future: _title,
            builder: (context, snapshot) {
              if (snapshot.hasError) return const Text('Could not load title');
              if (!snapshot.hasData) return const CircularProgressIndicator();
              return Padding(
                padding: const EdgeInsets.all(24),
                child: Text(snapshot.data!, textAlign: TextAlign.center),
              );
            },
          ),
        ),
      );
}
```

### Line-by-line explanation

- `initState` stores the future once.
- `_load` sends GET to `/posts/1`.
- It checks for HTTP 200 before decoding JSON.
- The response map's `title` becomes a `String`.
- `FutureBuilder` renders loading/error/success.
- Padding and centered text make the response readable.

### Widget tree

```text
MaterialApp -> GetPostPage -> Scaffold -> FutureBuilder
                                       ├-> spinner
                                       ├-> error
                                       └-> title
```

### Execution flow

```text
initState -> GET -> response -> status check -> decode -> title -> UI
```

### ASCII UI preview

```text
+---------------------------+
|   First GET request       |
+---------------------------+
| sunt aut facere...        |
|                           |
+---------------------------+
```

### Example explanation

GET usually has no body; query values belong in the URI. Storing the future prevents accidental duplicate calls during rebuild. If status validation is removed, server error JSON may be parsed as a post and fail confusingly.

**Common mistakes:** Do not make a state-changing request using GET or include private data in its URL.

**Professional use case:** Fetch profile, product, or feed data for display.

# Complete Example 2

Refresh by replacing the future from an event:

```dart
setState(() {
  _title = _load();
});
```

Call this from a refresh action, not from `build`.

# Complete Example 3

Create a reusable service:

```dart
Future<Post> fetchPost(http.Client client, int id) async { ... }
```

Injecting the client simplifies tests and keeps screen code focused on UI.

# Advanced Example

Add conditional GET with ETag/If-None-Match if the backend supports it. Handle 304 without trying to parse a body; use the validated cached representation.

# Under the Hood

The HTTP client performs URI resolution and transport asynchronously. `FutureBuilder` subscribes to a future and schedules builds as its connection state changes; it does not perform the request itself.

# Performance Notes

- Avoid calling GET on every rebuild.
- Reuse clients and cache appropriately.
- Paginate lists and request only needed fields.
- Show cached data while refreshing if UX benefits.

# Best Practices

- Use HTTPS and URI builders.
- Check status and payload shape.
- Add reasonable timeouts.
- Make retry explicit.
- Keep request in service/repository.

# Common Mistakes

- Recreating future in build.
- Ignoring response status.
- Force-casting unexpected JSON.
- Using GET for mutation.

# Debugging

- Verify URI and method.
- Check HTTP status separately from exceptions.
- Inspect sanitized body and content type.
- Test empty, malformed, and slow responses.

# Interview Questions

1. Is GET safe? By convention, it should not change server state.
2. Can GET be cached? Often yes, subject to headers/API behavior.
3. Where do filters go? Usually query parameters.
4. Why store Future in state? To avoid issuing duplicate requests on rebuild.

# Try It Yourself

1. Add a retry button.
2. Show a 404-specific message.
3. Add a timeout.
4. Map response to a typed model.

# Challenge Project

Build a GET-driven directory page with search parameters, loading/error/empty states, pagination, and mocked-client tests.

# Quick Revision Sheet

- GET retrieves resource representations.
- Construct URI safely.
- Check status before decoding.
- Keep Future stable across rebuilds.
- GET should not mutate server state.

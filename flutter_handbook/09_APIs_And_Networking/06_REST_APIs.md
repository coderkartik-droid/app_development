# Introduction

REST (Representational State Transfer) is an architectural style for networked APIs. REST-style APIs commonly expose resources through URLs and use HTTP methods to operate on them.

# Why this concept exists

Resource-oriented conventions make APIs easier to discover and compose. A consistent `/users/{id}/tasks` design is clearer than unrelated action names for every operation.

# Real-world analogy

A library catalogue assigns each book a stable address. You can retrieve, replace, update, or remove a particular book using the address and an appropriate action.

# Internal Working

A REST-style interaction identifies a resource with a URI and sends an HTTP method. Representations (often JSON) describe resource state. Statelessness means each request carries the context needed for that request rather than relying on hidden client session state, though servers may still maintain resources and tokens.

REST is a set of architectural constraints, not a synonym for “JSON over HTTP.” Real APIs often use REST-like conventions without satisfying every formal constraint.

# Syntax

```text
GET    /v1/tasks
POST   /v1/tasks
GET    /v1/tasks/{taskId}
PATCH  /v1/tasks/{taskId}
DELETE /v1/tasks/{taskId}
```

# Complete Example 1

This runnable Flutter screen fetches a list of resource representations from a demo API. Add `http`. The sample service provides posts rather than a production REST contract.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const RestApp());

class RestApp extends StatelessWidget {
  const RestApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: RestPage());
}

class RestPage extends StatefulWidget {
  const RestPage({super.key});

  @override
  State<RestPage> createState() => _RestPageState();
}

class _RestPageState extends State<RestPage> {
  late Future<List<String>> _titles;

  @override
  void initState() {
    super.initState();
    _titles = _loadTitles();
  }

  Future<List<String>> _loadTitles() async {
    final response = await http.get(
      Uri.https('jsonplaceholder.typicode.com', '/posts'),
    );
    if (response.statusCode != 200) {
      throw Exception('HTTP ${response.statusCode}');
    }
    final rows = jsonDecode(response.body) as List<dynamic>;
    return rows
        .take(10)
        .map((row) => (row as Map<String, dynamic>)['title'] as String)
        .toList();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('REST resources')),
        body: FutureBuilder<List<String>>(
          future: _titles,
          builder: (context, snapshot) {
            if (snapshot.hasError) return const Center(child: Text('Could not load'));
            if (!snapshot.hasData) {
              return const Center(child: CircularProgressIndicator());
            }
            return ListView.builder(
              itemCount: snapshot.data!.length,
              itemBuilder: (context, index) =>
                  ListTile(title: Text(snapshot.data![index])),
            );
          },
        ),
      );
}
```

### Line-by-line explanation

- `_titles` holds the request future so builds do not launch duplicate calls.
- `_loadTitles` sends GET to the collection resource.
- Non-success status throws before JSON parsing.
- The JSON list is limited to ten for display and each map's title is extracted.
- `FutureBuilder` handles error, loading, and success states.
- `ListView.builder` creates rows efficiently.

### Widget tree

```text
MaterialApp -> RestPage -> Scaffold -> FutureBuilder
                                     ├-> loading
                                     ├-> error
                                     └-> ListView -> ListTile*
```

### Execution flow

```text
GET collection URI -> server returns representations
                    -> decode list -> extract fields
                    -> FutureBuilder -> list UI
```

### ASCII UI preview

```text
+---------------------------+
|       REST resources      |
+---------------------------+
| sunt aut facere...        |
| qui est esse...           |
| ea molestias quasi...     |
+---------------------------+
```

### Example explanation

The endpoint is a collection URI. A real API might paginate and return a typed envelope rather than a raw list. The sample extracts only the title to keep the UI focused.

If the future is rebuilt each time, network requests can repeat. If status checking is removed, an error payload can be mistaken for a resource representation.

**Common mistakes:** Do not claim a service is fully REST simply because it uses HTTP and JSON.

**Professional use case:** Resource APIs for users, orders, tasks, and catalogs.

# Complete Example 2

Create a resource:

```dart
final response = await http.post(
  Uri.https('api.example.com', '/v1/tasks'),
  headers: {'content-type': 'application/json'},
  body: jsonEncode({'title': title}),
);
```

Check whether the API returns `201 Created`, a representation, and/or a `Location` header.

# Complete Example 3

Read one resource using its identifier:

```dart
final uri = Uri.https('api.example.com', '/v1/tasks/$taskId');
```

Identifiers should be URI-safe. If user-controlled, place them through URI path segment construction or validate/encode appropriately.

# Advanced Example

Design resource names as nouns, use consistent pluralization and versioning, represent errors consistently, and use pagination/filter parameters for collections. Support conditional requests and ETags where beneficial. Document idempotency and concurrency behavior.

# Under the Hood

The server maps URI and method to a handler, applies authorization, validates data, performs operations, and serializes a representation. The client maps the response to typed models. REST constraints include stateless interactions and cacheability, but practical APIs vary.

# Performance Notes

- Paginate collection endpoints.
- Request only necessary fields where supported.
- Cache safe resources with correct validators.
- Avoid N+1 calls.

# Best Practices

- Model resources and relationships clearly.
- Use HTTP semantics and status codes consistently.
- Version contracts.
- Parameterize user input on the server.
- Keep response schemas documented.

# Common Mistakes

- Verb-heavy endpoints for simple resource CRUD.
- Inconsistent IDs and pluralization.
- Ignoring pagination.
- Calling a JSON endpoint “REST” without considering semantics.
- Using HTTP status inconsistently.

# Debugging

- Verify URI, method, version, and content headers.
- Inspect error representation and status.
- Check pagination links/cursors.
- Confirm route IDs are encoded correctly.

# Interview Questions

1. What is a resource? A domain entity/addressable concept exposed by an API.
2. What does stateless mean? Each request contains context required for processing.
3. Is JSON required by REST? No.
4. Why use nouns in REST paths? Paths identify resources; methods express common actions.

# Try It Yourself

1. Design CRUD paths for books and authors.
2. Add pagination parameters to a collection.
3. Define status/body for create and not-found cases.
4. Identify REST-like behavior in a public API.

# Challenge Project

Design a REST contract for a personal tasks API with collection, item, search, pagination, authentication, and error response schemas.

# Quick Revision Sheet

- REST organizes around resources and representations.
- HTTP method + URI describes the operation.
- JSON is common, not mandatory.
- Use consistent status, pagination, versioning, and errors.

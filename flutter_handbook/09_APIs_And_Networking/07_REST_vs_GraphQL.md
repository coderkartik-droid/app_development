# Introduction

REST and GraphQL are two ways for clients to communicate with APIs. REST commonly exposes multiple resource endpoints; GraphQL commonly exposes one endpoint where a client submits a query describing requested fields.

# Why this concept exists

Different products have different data needs. REST offers straightforward resource semantics and HTTP tooling. GraphQL can let clients request related fields in one tailored operation, at the cost of query and caching complexity.

# Real-world analogy

REST is ordering from several menu sections by item. GraphQL is giving the kitchen a precise custom order listing exactly the ingredients and portions you want.

# Internal Working

With REST, the server maps method and path to endpoint logic and returns a representation. With GraphQL, the client sends a query document; the server validates it against a schema and resolves requested fields. A GraphQL response may contain both `data` and `errors`, even with HTTP 200.

# Syntax

REST:

```text
GET /users/7
GET /users/7/posts
```

GraphQL:

```graphql
query {
  user(id: "7") {
    name
    posts { title }
  }
}
```

# Complete Example 1

This runnable Flutter app fetches a REST resource and displays selected fields. Add `http`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const ApiStyleApp());

class ApiStyleApp extends StatelessWidget {
  const ApiStyleApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: ApiStylePage());
}

class ApiStylePage extends StatefulWidget {
  const ApiStylePage({super.key});

  @override
  State<ApiStylePage> createState() => _ApiStylePageState();
}

class _ApiStylePageState extends State<ApiStylePage> {
  late Future<String> _name;

  @override
  void initState() {
    super.initState();
    _name = _fetchName();
  }

  Future<String> _fetchName() async {
    final response = await http.get(
      Uri.https('jsonplaceholder.typicode.com', '/users/1'),
    );
    if (response.statusCode != 200) {
      throw Exception('HTTP ${response.statusCode}');
    }
    final map = jsonDecode(response.body) as Map<String, dynamic>;
    return map['name'] as String;
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('REST example')),
        body: Center(
          child: FutureBuilder<String>(
            future: _name,
            builder: (context, snapshot) {
              if (snapshot.hasError) return const Text('Request failed');
              if (!snapshot.hasData) return const CircularProgressIndicator();
              return Text('User: ${snapshot.data}');
            },
          ),
        ),
      );
}
```

### Line-by-line explanation

- The client requests a single user resource from a REST-style path.
- `_fetchName` checks status, decodes the JSON object, and selects the name.
- `FutureBuilder` shows progress, error, or the returned name.
- Storing `_name` in state prevents duplicate calls during rebuild.

### Widget tree

```text
MaterialApp -> ApiStylePage -> Scaffold -> FutureBuilder<String>
                                      ├-> spinner
                                      ├-> error
                                      └-> user name
```

### Execution flow

```text
GET /users/1 -> server representation -> decode user -> display name
```

### ASCII UI preview

```text
+---------------------------+
|       REST example        |
+---------------------------+
| User: Leanne Graham       |
+---------------------------+
```

### Example explanation

The client receives the endpoint's defined user representation, even though it only needs a name. A REST API may support field selection or linked endpoints, but its contract determines that.

If status handling is removed, non-success bodies may be decoded as user data. A GraphQL client would instead send a query document and inspect GraphQL-level `errors`.

**Common mistakes:** Choosing based only on popularity ignores caching, tooling, server ownership, and client requirements.

**Professional use case:** REST is a strong fit for stable resource CRUD; GraphQL can fit clients with varied related-data needs.

# Complete Example 2

A GraphQL request sends a query in JSON:

```dart
final response = await http.post(
  Uri.https('api.example.com', '/graphql'),
  headers: {'content-type': 'application/json'},
  body: jsonEncode({
    'query': 'query { user(id: "7") { name } }',
  }),
);
```

The endpoint and query schema are service-specific. Inspect both HTTP status and the response's `errors` field.

# Complete Example 3

For a REST list/detail flow, fetch a collection and then details as needed. For GraphQL, request related fields in one operation if the schema allows it. The trade-off is fewer round trips versus query complexity and server resolver cost.

# Advanced Example

Compare normalized client caching, authorization at field/resolver level, query complexity limits, pagination, and observability. GraphQL does not eliminate N+1 problems on the server; DataLoader-like batching and query budgets may be needed. REST caching benefits from standardized URI/method semantics.

# Under the Hood

REST endpoint routing uses HTTP method/path; GraphQL parses and validates a query against a schema and invokes field resolvers. GraphQL transport commonly uses HTTP POST, but protocol and subscription behavior depend on implementation.

# Performance Notes

- Count network round trips and payload size.
- Consider cacheability and normalization.
- Limit GraphQL query depth/complexity.
- Avoid requesting unused fields.
- Measure server resolver and client decoding cost.

# Best Practices

- Choose based on product/team/API needs.
- Generate typed clients/models where practical.
- Handle partial GraphQL errors.
- Document REST status/error contracts.
- Secure authorization at every resource or field.

# Common Mistakes

- Assuming GraphQL automatically makes requests faster.
- Assuming REST requires one endpoint per screen.
- Ignoring GraphQL partial errors.
- Overfetching fields or creating deeply expensive queries.
- Comparing only number of endpoints.

# Debugging

- REST: inspect URI, method, status, headers, response.
- GraphQL: inspect query, variables, HTTP status, `data`, and `errors`.
- Check authorization and pagination.
- Measure actual payload and resolver latency.

# Interview Questions

1. What is the key client-query difference? GraphQL clients specify fields; REST endpoints define representations.
2. Does GraphQL always use one URL? Commonly one endpoint, but implementation can vary.
3. Can GraphQL return HTTP 200 with errors? Yes; inspect GraphQL response errors.
4. Which is easier to cache with standard HTTP semantics? Often REST, though GraphQL caching is possible with additional strategies.

# Try It Yourself

1. Write a REST endpoint set for a profile screen.
2. Write a GraphQL query requesting the same profile and posts.
3. Compare payload sizes.
4. Design handling for partial GraphQL errors.

# Challenge Project

For an ecommerce product screen, compare REST and GraphQL contracts for product, variants, reviews, and stock. Choose one and justify caching, pagination, and authorization.

# Quick Revision Sheet

- REST: resources addressed by URIs and HTTP methods.
- GraphQL: schema and client-selected fields/operations.
- GraphQL may return partial data plus errors.
- Neither automatically solves performance or security.
- Compare caching, client needs, complexity, and team support.

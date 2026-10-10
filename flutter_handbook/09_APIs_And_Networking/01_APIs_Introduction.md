# Introduction

An API (Application Programming Interface) is a defined way for one program to request information or actions from another. Flutter apps commonly call web APIs to load weather, news, account details, maps, or product catalogs.

A network call has two sides: the client sends a request, and a server returns a response. The response may contain JSON, a file, or an error. The app turns that result into Dart values and then into UI.

# Why this concept exists

Apps often need data that changes frequently or is shared among users. A server provides a central source of truth, while an API gives mobile clients a predictable contract for retrieving and changing that data.

# Real-world analogy

An API is like ordering at a café. The menu defines valid requests, the order is sent to the kitchen, and the server returns the prepared result or explains why it could not fulfill the order.

# Internal Working

Flutter code runs in Dart. A networking package opens or reuses a connection and sends bytes over platform networking. The server receives the request, authenticates and processes it, then sends a status code, headers, and response body. Dart receives the response asynchronously, decodes the body (often JSON), maps it to models, and updates state. Flutter then builds the relevant widgets.

Request lifecycle:

```text
User action -> Dart client -> DNS/TCP/TLS/HTTP -> server
             <- response bytes <- status/headers/body
             -> decode -> model -> state -> widgets
```

Never perform a long network request synchronously in `build`. Model loading, success, empty, and error states; requests can take time or fail.

# Syntax

```dart
final response = await http.get(Uri.https('api.example.com', '/v1/items'));
if (response.statusCode == 200) {
  final json = jsonDecode(response.body);
}
```

This uses the `http` package; add it to the app's dependencies. `Uri.https` constructs and escapes the request URI safely.

# Complete Example 1

This runnable app fetches a public JSON placeholder post and displays title/body. Add the `http` package before running. The service is a demo API and should not be used as a production backend.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const ApiIntroApp());

class ApiIntroApp extends StatelessWidget {
  const ApiIntroApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: PostPage());
}

class Post {
  const Post({required this.title, required this.body});
  final String title;
  final String body;

  factory Post.fromJson(Map<String, dynamic> json) => Post(
        title: json['title'] as String,
        body: json['body'] as String,
      );
}

class PostPage extends StatefulWidget {
  const PostPage({super.key});

  @override
  State<PostPage> createState() => _PostPageState();
}

class _PostPageState extends State<PostPage> {
  late Future<Post> _post;

  @override
  void initState() {
    super.initState();
    _post = _fetchPost();
  }

  Future<Post> _fetchPost() async {
    final uri = Uri.https('jsonplaceholder.typicode.com', '/posts/1');
    final response = await http.get(uri).timeout(const Duration(seconds: 10));
    if (response.statusCode != 200) {
      throw Exception('Server returned ${response.statusCode}');
    }
    final decoded = jsonDecode(response.body) as Map<String, dynamic>;
    return Post.fromJson(decoded);
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('First API call')),
        body: Center(
          child: FutureBuilder<Post>(
            future: _post,
            builder: (context, snapshot) {
              if (snapshot.connectionState != ConnectionState.done) {
                return const CircularProgressIndicator();
              }
              if (snapshot.hasError) {
                return Text('Could not load post: ${snapshot.error}');
              }
              final post = snapshot.data!;
              return Padding(
                padding: const EdgeInsets.all(24),
                child: Column(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    Text(post.title, style: Theme.of(context).textTheme.titleLarge),
                    const SizedBox(height: 12),
                    Text(post.body),
                  ],
                ),
              );
            },
          ),
        ),
      );
}
```

### Line-by-line explanation

- `dart:convert` decodes JSON; Material provides UI; `http` sends the request.
- `Post` is a typed model. `fromJson` maps server keys into Dart fields.
- `initState` starts the request once rather than launching it on every build.
- `_fetchPost` constructs an HTTPS URI, makes an asynchronous GET, and applies a timeout.
- A non-200 status becomes an error instead of being decoded as valid content.
- JSON is decoded to a map, then converted into `Post`.
- `FutureBuilder` renders loading until completion, an error message on failure, or the post on success.
- `Padding`, `Column`, `Text`, and `SizedBox` present readable content.

### Widget tree

```text
MaterialApp -> PostPage -> Scaffold -> Center -> FutureBuilder
                                                    ├-> spinner
                                                    ├-> error Text
                                                    └-> post title/body
```

### Execution flow

```text
initState -> GET URI -> server status/body -> JSON decode -> Post
                                      |
                      FutureBuilder success/error/loading
                                      v
                                     UI
```

### ASCII UI preview

```text
+---------------------------+
|       First API call      |
+---------------------------+
|  sunt aut facere...       |
|                           |
|  quia et suscipit...      |
+---------------------------+
```

### Example explanation

The future is stored in state so unrelated rebuilds do not repeat the request. The model keeps JSON parsing out of presentation code. A timeout prevents waiting forever, while status validation avoids treating server errors as successful payloads.

If `_post = _fetchPost()` is moved into `build`, every rebuild can issue another request. If the status check is removed, an error response may fail later during parsing with a less useful message. If the loading branch is removed, the UI has no representation for an in-flight call.

**Common mistakes:** Do not put API keys in public client code; do not assume every successful HTTP response contains valid JSON or required fields.

**Professional use case:** A repository fetches typed remote data and exposes loading/error/success state to a screen.

# Complete Example 2

A typed client method should return a model rather than raw response text:

```dart
Future<Post> getPost(int id) async {
  final uri = Uri.https('api.example.com', '/posts/$id');
  final response = await http.get(uri);
  if (response.statusCode != 200) throw ApiException(response.statusCode);
  return Post.fromJson(jsonDecode(response.body) as Map<String, dynamic>);
}
```

`id` is part of the path; `Uri.https` should still be preferred when assembling user-provided values. Validate response shape and catch/translate package exceptions at the repository boundary.

# Complete Example 3

POST sends data to create a resource:

```dart
final response = await http.post(
  Uri.https('api.example.com', '/posts'),
  headers: {'content-type': 'application/json'},
  body: jsonEncode({'title': 'Hello'}),
);
```

The content type tells the server how to interpret the body. Check the API's expected schema and returned status rather than assuming creation succeeded.

# Advanced Example

Production clients need centralized base URLs, authentication, request IDs, timeouts, cancellation policy, serialization, and consistent error mapping. Avoid leaking personally identifiable data in logs. Consider retries only for safe/idempotent operations or requests protected by idempotency keys.

# Under the Hood

An HTTP client serializes the request line, headers, and body, then sends them through the platform's networking stack. The server responds with status, headers, and bytes. JSON decoding is a separate step; successful transport does not imply a successful application response.

# Performance Notes

- Reuse an `http.Client` for multiple requests and close it when its owner is disposed.
- Paginate large responses.
- Avoid blocking the UI isolate with huge decoding workloads.
- Cache only with freshness and invalidation rules.
- Use timeouts and avoid duplicate requests from rebuilds.

# Best Practices

- Use HTTPS and typed models.
- Separate API client, repository, and UI.
- Handle status codes, malformed data, and timeouts distinctly.
- Keep secrets server-side where possible.
- Test with mocked clients.

# Common Mistakes

- Starting network calls inside `build`.
- Ignoring status codes.
- Force-casting untrusted JSON without validation.
- Sending secrets in query strings.
- Retrying non-idempotent writes blindly.

# Debugging

- Inspect method, URI, request headers, status, and sanitized body.
- Reproduce with a known API tool or mock server.
- Distinguish DNS, TLS, timeout, HTTP, and JSON parsing failures.
- Never log bearer tokens or private response fields.

# Interview Questions

1. What does an API provide? A contract for software components to exchange data or actions.
2. Does HTTP 200 mean the JSON is valid? No.
3. Why use `FutureBuilder` or state management? To model asynchronous loading/result states.
4. Why use HTTPS? It protects transport confidentiality and integrity.
5. What layers commonly separate networking? Client, repository, and presentation/state.

# Try It Yourself

1. Add a retry button to the post screen.
2. Display a friendly empty state for a 404.
3. Move `_fetchPost` into an API service.
4. Test malformed JSON and slow network behavior.

# Challenge Project

Build a small public-profile viewer with typed models, retryable loading/error state, timeout handling, and a repository abstraction. Include tests with mocked responses.

# Quick Revision Sheet

- Request = method + URI + headers + optional body.
- Response = status + headers + body.
- Decode JSON only after status/content checks.
- Model network state explicitly.
- Keep requests out of `build`; use timeouts and tests.

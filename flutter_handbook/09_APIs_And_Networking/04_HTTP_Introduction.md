# Introduction

HTTP (Hypertext Transfer Protocol) is the request/response protocol used by most web APIs. HTTPS is HTTP carried through TLS-protected transport.

An HTTP request includes a method, target URI, headers, and sometimes a body. A response includes a status code, headers, and optional body.

# Why this concept exists

Clients and servers need common rules for describing operations and returning results. HTTP defines those rules independently of Flutter or Dart.

# Real-world analogy

An HTTP request is a structured form sent to a service desk: the action is the method, the destination is the URI, extra instructions are headers, and details are the body. The response includes an outcome code and a reply.

# Internal Working

The client constructs a URI, serializes method/headers/body, and sends the request over a connection. The server routes it, applies application logic, and returns a status and representation. The networking call is asynchronous; Flutter widgets do not wait synchronously for the server.

# Syntax

```dart
final response = await http.get(
  Uri.https('api.example.com', '/v1/items'),
  headers: {'accept': 'application/json'},
);
```

# Complete Example 1

This app requests JSON and shows the HTTP status and content type. Add the `http` dependency.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const HttpIntroApp());

class HttpIntroApp extends StatelessWidget {
  const HttpIntroApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: HttpPage());
}

class HttpPage extends StatefulWidget {
  const HttpPage({super.key});

  @override
  State<HttpPage> createState() => _HttpPageState();
}

class _HttpPageState extends State<HttpPage> {
  String _result = 'Tap to send request';

  Future<void> _send() async {
    setState(() => _result = 'Waiting for response...');
    try {
      final response = await http.get(
        Uri.https('jsonplaceholder.typicode.com', '/posts/1'),
        headers: {'accept': 'application/json'},
      );
      if (!mounted) return;
      setState(() {
        _result = 'Status ${response.statusCode}\n'
            'Content-Type: ${response.headers['content-type'] ?? 'unknown'}';
      });
    } catch (error) {
      if (!mounted) return;
      setState(() => _result = 'Request failed');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('HTTP request/response')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_result, textAlign: TextAlign.center),
              const SizedBox(height: 16),
              FilledButton(onPressed: _send, child: const Text('GET')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- Imports provide UI and the HTTP package.
- `_send` displays an in-progress message, then sends a GET to an HTTPS URI.
- The `Accept` header tells the server the preferred representation.
- The response status and content-type header are shown.
- A catch handles transport exceptions separately from returned status codes.
- `mounted` guards asynchronous UI updates.
- The button is the user-triggered request source.

### Widget tree

```text
MaterialApp -> HttpPage -> Scaffold -> Center -> Column
                                              ├-> response text
                                              └-> GET button
```

### Execution flow

```text
Button -> request method/URI/header -> server
       <- response status/header/body <- server
       -> update state -> render result
```

### ASCII UI preview

```text
+---------------------------+
| HTTP request/response     |
+---------------------------+
| Status 200                |
| Content-Type: application |
| [          GET          ] |
+---------------------------+
```

### Example explanation

The response is more than its body: status and headers explain how the server handled the request and how to interpret its representation. A 404 is still a valid HTTP response, not a connection failure.

If the `Accept` header is removed, the server chooses its default representation. If exception handling is removed, DNS/TLS/timeouts can surface as unhandled errors. This sample does not decode the body because it focuses on HTTP structure.

**Common mistakes:** Do not infer success from receiving any response; check status according to the endpoint contract.

**Professional use case:** Shared HTTP client methods that normalize headers, timeouts, and error mapping.

# Complete Example 2

A response body can be JSON:

```dart
final json = jsonDecode(response.body);
```

Decode only after checking status and content type where the API contract requires JSON.

# Complete Example 3

A request body can declare its format:

```dart
headers: {'content-type': 'application/json'},
body: jsonEncode({'title': 'Example'}),
```

The content type describes the outgoing body, while `Accept` describes the desired response.

# Advanced Example

HTTP semantics include caching, conditional requests, redirects, content negotiation, and idempotency. A client should respect API-specific contracts and avoid automatically retrying unsafe operations without a deduplication mechanism.

# Under the Hood

An HTTP client writes protocol data over sockets and handles connection state. HTTPS uses TLS certificate validation. The returned response object exposes status, headers, and body; decoding into domain objects remains application code.

# Performance Notes

- Reuse clients and connections.
- Set timeouts and response size limits where supported.
- Avoid redundant requests.
- Compress and paginate large payloads.
- Use streaming for large downloads.

# Best Practices

- Use `Uri` builders.
- Set correct `Accept` and `Content-Type`.
- Validate status and body.
- Use HTTPS.
- Separate transport errors from HTTP errors.

# Common Mistakes

- Confusing request `Content-Type` with `Accept`.
- Ignoring status codes.
- Sending GET bodies without API support.
- Building URLs by unsafe concatenation.
- Treating all failures as the same error.

# Debugging

- Inspect method, URI, headers, status, and content type.
- Confirm endpoint expects the chosen method/body.
- Check TLS and redirects.
- Verify the response size and encoding.

# Interview Questions

1. What are the parts of an HTTP request? Method, URI, headers, and optional body.
2. What are the parts of a response? Status, headers, and optional body.
3. What does `Accept` mean? Preferred response representation.
4. What does `Content-Type` mean? Format of the message body.
5. Is 404 a transport error? No; it is an HTTP response status.

# Try It Yourself

1. Show response body length and content type.
2. Compare 200, 404, and offline behavior.
3. Add timeout handling.
4. Send JSON with both content headers.

# Challenge Project

Create a diagnostic screen that can make GET and POST requests to a mock endpoint and display sanitized request/response metadata.

# Quick Revision Sheet

- HTTP request: method, URI, headers, body.
- HTTP response: status, headers, body.
- HTTPS = HTTP protected by TLS.
- `Accept` is response preference; `Content-Type` describes body.
- Check status and handle transport exceptions.

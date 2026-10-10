# Introduction

Flutter uses HTTP requests to communicate with DRF endpoints. Requests include method, URL, headers, optional body, and timeout/retry policy.

# Why this concept exists

A consistent transport layer prevents widgets from duplicating URL construction, authentication headers, and error handling.

# Real-world analogy

The HTTP client is a courier: it carries a labeled package to the server and brings back a receipt.

# Internal Working

Flutter's HTTP package returns a Future. It sends bytes via the platform network stack; DRF parses request bytes and emits an HTTP response. The Future completes with a response or throws a transport exception.

# Data Flow

```text
Flutter model -> JSON encode -> HTTP request -> DRF parse/handle
-> response JSON -> decode/map -> UI state
```

# Complete Example 1

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class ApiClient {
  ApiClient({http.Client? client}) : _client = client ?? http.Client();
  final http.Client _client;

  Future<Map<String, dynamic>> getJson(Uri uri) async {
    final response = await _client
        .get(uri, headers: {'Accept': 'application/json'})
        .timeout(const Duration(seconds: 15));
    if (response.statusCode < 200 || response.statusCode >= 300) {
      throw StateError('HTTP ${response.statusCode}: ${response.body}');
    }
    return jsonDecode(response.body) as Map<String, dynamic>;
  }

  void close() => _client.close();
}
```

The optional injected client supports tests; `Accept` requests JSON; timeout bounds waiting; status is checked before decoding. Replace `StateError` with typed API exceptions that safely retain status and parsed validation details. Close owned clients with their lifecycle.

# Complete Example 2

DRF endpoint:

```python
from rest_framework.response import Response
from rest_framework.views import APIView
from rest_framework.permissions import AllowAny

class HealthView(APIView):
    permission_classes = [AllowAny]

    def get(self, request):
        return Response({"status": "ok"})
```

DRF constructs a response from primitive data, then negotiates/renders JSON. This endpoint is intentionally public and minimal. Mount its URL under the same base path the Flutter client uses.

# Complete Example 3

```dart
final uri = Uri.https('api.example.com', '/api/v1/products', {
  'search': 'desk lamp',
  'page': '2',
});
final response = await http.get(uri);
```

`Uri.https` encodes query parameters safely rather than concatenating user text. Check status and parse the body in real code. Never put passwords or tokens in query strings, where they may be logged.

# Advanced Example

Add centralized interceptors/middleware for bearer headers, correlation IDs, retry policy, and error decoding. Retry only safe/idempotent operations or writes protected by an idempotency key.

# Under the Hood

HTTP methods carry semantics; headers describe content and credentials; response status communicates outcome. Network errors and HTTP error statuses are distinct failure types.

# Performance Notes

Reuse clients, bound timeouts, compress suitable payloads, and avoid excessive concurrent requests.

# Security Notes

Use HTTPS, keep tokens out of URLs/logs, and do not disable certificate validation.

# Best Practices

- Centralize transport.
- Encode query/body data with APIs, not string concatenation.
- Distinguish transport, status, and parsing failures.
- Inject clients for deterministic tests.

# Common Mistakes

- Decoding error HTML as JSON.
- Ignoring non-2xx status.
- Retrying POST blindly.

# Debugging

Inspect URL/method, headers, body, status, response content type, TLS, timeout, and decoded model fields.

# Interview Questions

1. Difference between HTTP failure status and network exception?
2. Why use Uri query parameters?
3. When is retry safe?

# Practice Exercises

1. Build an injectable client with timeout.
2. Map 400 validation errors separately from 500.
3. Test URL encoding for search text.

# Chapter Summary

Centralized HTTP handling gives Flutter a reliable boundary to DRF. Correctly encode requests, inspect status before decoding, and define safe timeout/retry behavior.

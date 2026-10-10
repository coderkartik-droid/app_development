# Introduction

API error handling turns failures from transport, HTTP status, decoding, and application rules into understandable states. A robust client preserves enough detail for developers while giving users clear next steps.

# Why this concept exists

Requests fail for different reasons. A validation error needs correction; a 401 may need authentication; a timeout may be retryable; malformed JSON may indicate a contract problem.

# Real-world analogy

A courier can fail because the address is wrong, the road is closed, the recipient refused delivery, or the parcel is damaged. Each cause needs a different response.

# Internal Working

Classify errors at the API/repository boundary: network/timeout, HTTP status, parsing/contract, and domain validation. Preserve request ID and cause for diagnostics, but do not expose stack traces or secrets to users. UI should represent loading, success, empty, and recoverable failure.

# Syntax

```dart
try {
  final response = await client.get(uri);
  if (response.statusCode >= 400) throw ApiException(response.statusCode);
} on TimeoutException {
  // Offer retry.
} on FormatException {
  // Report unexpected payload.
}
```

# Complete Example 1

This app fetches a demo post and shows distinct loading, success, and failure states. Add `http`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const ApiErrorsApp());

class ApiErrorsApp extends StatelessWidget {
  const ApiErrorsApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: ApiErrorsPage());
}

class ApiErrorsPage extends StatefulWidget {
  const ApiErrorsPage({super.key});

  @override
  State<ApiErrorsPage> createState() => _ApiErrorsPageState();
}

class _ApiErrorsPageState extends State<ApiErrorsPage> {
  String? _title;
  String? _error;
  bool _loading = false;

  Future<void> _load() async {
    setState(() {
      _loading = true;
      _error = null;
    });
    try {
      final response = await http
          .get(Uri.https('jsonplaceholder.typicode.com', '/posts/1'))
          .timeout(const Duration(seconds: 8));
      if (response.statusCode != 200) {
        throw HttpException('Unexpected status ${response.statusCode}');
      }
      final json = jsonDecode(response.body) as Map<String, dynamic>;
      final title = json['title'];
      if (title is! String) throw const FormatException('Missing title');
      if (!mounted) return;
      setState(() => _title = title);
    } on FormatException {
      if (!mounted) return;
      setState(() => _error = 'The server response was unexpected.');
    } catch (error) {
      if (!mounted) return;
      setState(() => _error = 'Could not load data. Please retry.');
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('API error states')),
        body: Center(
          child: _loading
              ? const CircularProgressIndicator()
              : _error != null
                  ? Column(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        Text(_error!),
                        FilledButton(onPressed: _load, child: const Text('Retry')),
                      ],
                    )
                  : Column(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        Text(_title ?? 'No data loaded'),
                        FilledButton(onPressed: _load, child: const Text('Load')),
                      ],
                    ),
        ),
      );
}

class HttpException implements Exception {
  const HttpException(this.message);
  final String message;
}
```

### Line-by-line explanation

- The request is bounded by a timeout.
- Non-200 status becomes a typed local exception.
- JSON decoding and explicit `String` validation detect malformed payloads.
- `FormatException` receives a contract-focused message.
- Other failures become a recoverable retry message.
- `finally` always clears loading.
- The UI includes a retry action only on failure.
- `HttpException` carries diagnostic context; production may use a richer typed hierarchy.

### Widget tree

```text
Scaffold → Center
          ├→ loading spinner
          ├→ error + Retry
          └→ title + Load
```

### Execution flow

```text
Request -> timeout/status/body parse/model validation
         ├-> success -> display
         ├-> malformed -> response error
         └-> transport/other -> retryable message
```

### ASCII UI preview

```text
+---------------------------+
|     API error states      |
+---------------------------+
| Could not load data.      |
| Please retry.             |
| [          Retry        ] |
+---------------------------+
```

### Example explanation

The code checks the payload field rather than trusting a cast alone. User-facing text is actionable and does not expose an exception stack. Internal diagnostics should still retain sanitized cause and request ID.

If `finally` is removed, loading may remain true. If the broad catch is removed, transport errors may escape; if it catches everything without preserving diagnostics, bugs become hidden.

**Common mistakes:** Do not retry validation failures automatically.

# Complete Example 2

Map status codes into domain errors:

```dart
switch (response.statusCode) {
  case 401: throw const SessionExpiredException();
  case 404: throw const NotFoundException();
  case 429: throw const RateLimitedException();
}
```

The repository can then present consistent behavior across screens.

# Complete Example 3

Handle an API error body only if it matches the documented schema:

```dart
final message = errorJson['message'];
```

Sanitize server messages before showing them; internal details may reveal implementation or sensitive data.

# Advanced Example

Model retryability separately from severity. Use exponential backoff with jitter for transient network/5xx failures, respect Retry-After, and cap attempts. Do not retry unsafe writes unless idempotency is guaranteed.

# Under the Hood

Transport exceptions occur before a normal response exists. HTTP errors have status and headers. JSON parsing errors happen after transport succeeds. Domain errors may be encoded in a successful HTTP response depending on API design.

# Performance Notes

- Avoid repeatedly parsing large error bodies.
- Keep stack traces out of user UI.
- Prevent concurrent duplicate retries.
- Record request IDs and timing.

# Best Practices

- Use typed errors.
- Keep user message and developer diagnostics separate.
- Make retry explicit and bounded.
- Preserve previous content when useful.
- Handle empty success responses.

# Common Mistakes

- Catching and ignoring every exception.
- Treating all failures as offline.
- Retrying every status.
- Showing raw server body.
- Losing the underlying error cause.

# Debugging

- Identify stage: transport, status, decode, model, or domain.
- Capture sanitized method/path/status/request ID.
- Reproduce with fixtures and mock clients.
- Check retry loops and loading cleanup.

# Interview Questions

1. What error categories should an API client distinguish? Transport, HTTP, parsing, and domain.
2. Why separate user message from diagnostics? Avoid exposing internals while retaining debug detail.
3. Which errors are retryable? Usually transient ones, depending on method/idempotency and server guidance.
4. Why handle 204 separately? It has no body to parse.

# Try It Yourself

1. Add typed 401/404/429 errors.
2. Add a timeout-specific message.
3. Test malformed response JSON.
4. Add bounded retry to a safe GET.

# Challenge Project

Build a shared error mapper and test network failure, timeout, 400, 401, 404, 429, 500, malformed JSON, and empty success.

# Quick Revision Sheet

- Separate transport, HTTP, parsing, and domain errors.
- Model loading/success/empty/error.
- Preserve diagnostics; sanitize user messages.
- Retry only when safe and bounded.
- Always clear loading state.

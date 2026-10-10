# Introduction

An HTTP status code summarizes the outcome of a request. Flutter clients should interpret it according to the endpoint contract rather than treating every response as success or failure in the same way.

# Why this concept exists

The body alone may not tell the complete story. Status codes distinguish successful reads, accepted work, missing resources, invalid requests, authorization failures, rate limits, and server problems.

# Real-world analogy

A delivery receipt might say delivered, address unknown, payment required, or delayed. The short status guides what to do next.

# Internal Working

Status codes are grouped by first digit: 1xx informational, 2xx success, 3xx redirection, 4xx client/request issue, and 5xx server failure. APIs may define specific body schemas and retry guidance for each code.

# Syntax

```dart
if (response.statusCode >= 200 && response.statusCode < 300) {
  // Successful HTTP response; validate its expected body.
}
```

# Complete Example 1

This runnable UI maps a demo endpoint's response into simple messages. Add `http`.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const StatusCodesApp());

class StatusCodesApp extends StatelessWidget {
  const StatusCodesApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: StatusPage());
}

class StatusPage extends StatefulWidget {
  const StatusPage({super.key});

  @override
  State<StatusPage> createState() => _StatusPageState();
}

class _StatusPageState extends State<StatusPage> {
  String _message = 'Not checked';

  Future<void> _check() async {
    try {
      final response = await http.get(
        Uri.https('jsonplaceholder.typicode.com', '/posts/1'),
      );
      final message = switch (response.statusCode) {
        >= 200 && < 300 => 'Success (${response.statusCode})',
        401 => 'Sign in is required',
        404 => 'Resource was not found',
        >= 500 => 'Server is temporarily unavailable',
        _ => 'Request returned ${response.statusCode}',
      };
      if (mounted) setState(() => _message = message);
    } catch (error) {
      if (mounted) setState(() => _message = 'Could not reach the server');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('HTTP status')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_message),
              FilledButton(onPressed: _check, child: const Text('Check')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `_check` sends a GET and switches on the returned integer code.
- A 2xx range is treated as HTTP success but still may need body validation.
- 401, 404, and 5xx have different user-level meaning.
- Other statuses remain visible without inventing a false result.
- A caught exception represents failure to obtain a normal response.
- `mounted` protects async updates.

### Widget tree

```text
MaterialApp -> StatusPage -> Scaffold -> Center -> Column(Text, Button)
```

### Execution flow

```text
Request -> response? -> map status to application meaning -> render
        \-> transport exception -> reachability message
```

### ASCII UI preview

```text
+---------------------------+
|        HTTP status        |
+---------------------------+
| Success (200)              |
| [          Check        ] |
+---------------------------+
```

### Example explanation

Status mapping creates clearer UX than showing raw codes. A successful status does not guarantee the body matches the expected JSON model. A caught network exception has no HTTP status to inspect.

If all non-200 results are called “offline,” 401 and 404 are misrepresented. If every 2xx is assumed to include a body, endpoints returning 204 can break parsing.

**Common mistakes:** Design response handling around the API contract, including empty-body success such as 204.

**Professional use case:** A centralized error mapper turns endpoint statuses into typed app-level outcomes.

# Complete Example 2

Typical status groups:

```text
200 OK, 201 Created, 204 No Content
400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 429 Too Many Requests
500 Internal Server Error, 503 Service Unavailable
```

These descriptions are conventional; inspect API-specific meaning and retry headers.

# Complete Example 3

Use `Retry-After` when a rate-limited service supplies it:

```dart
final delay = response.headers['retry-after'];
```

Interpret format according to HTTP specification/API behavior and apply a bounded retry policy.

# Advanced Example

Map transport, authentication, validation, conflict, rate limit, and server failures to distinct domain exceptions. Include correlation IDs in diagnostics, and avoid retrying 4xx validation errors. A 409 may require refreshing state; 429/503 may allow bounded backoff.

# Under the Hood

The server emits a three-digit code in the HTTP response. The client receives it independently of JSON decoding. Many HTTP client libraries return non-2xx responses normally; callers must inspect status explicitly.

# Performance Notes

- Avoid retry storms on 5xx or 429.
- Respect `Retry-After` where appropriate.
- Do not parse large error bodies unnecessarily.
- Keep status mapping centralized.

# Best Practices

- Handle expected statuses explicitly.
- Treat unknown codes safely.
- Parse error bodies using documented schemas.
- Separate HTTP status from transport exceptions.
- Respect rate limits and idempotency.

# Common Mistakes

- Assuming any returned response means success.
- Treating 401 and 403 identically without policy.
- Parsing body for 204.
- Retrying all errors.
- Showing raw server internals to users.

# Debugging

- Record method/path/status/request ID.
- Check response headers and sanitized error payload.
- Verify whether the client throws on non-2xx or returns normally.
- Distinguish no response from error status.

# Interview Questions

1. What does 2xx indicate? Successful HTTP processing, subject to endpoint semantics.
2. Difference between 401 and 403? Authentication required/failed versus authenticated but not permitted, by common convention.
3. Why can 204 break JSON parsing? It intentionally has no response body.
4. What is 429? Rate limiting; check retry guidance.

# Try It Yourself

1. Add special handling for 204.
2. Map 401 to session refresh and 403 to permission messaging.
3. Parse a documented error object.
4. Add bounded retry for 503.

# Challenge Project

Create a typed API error mapper with tests for transport exceptions, 400, 401, 403, 404, 409, 429, 500, and 204.

# Quick Revision Sheet

- 2xx success; 3xx redirect; 4xx request/client issue; 5xx server issue.
- HTTP errors still arrive as responses.
- Transport errors have no status.
- 204 has no body.
- Retry based on status, idempotency, and server guidance.

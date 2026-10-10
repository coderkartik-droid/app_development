# Introduction

Debugging APIs means locating the layer where a request failed: URI construction, DNS/TLS, transport, HTTP status, payload parsing, model mapping, or UI state.

# Why this concept exists

The same user-visible message (“failed to load”) can have many causes. A repeatable diagnostic process reduces guesswork and avoids exposing sensitive request data.

# Real-world analogy

When a parcel is late, you check the label, dispatch scan, route, destination, and receipt. You do not assume the truck is broken before checking the address.

# Internal Working

Trace:

```text
user action -> request builder -> network client -> server
            <- response/error <- decode/model <- state <- widget
```

Record method, sanitized path, status, timing, request ID, and error category. Redact tokens, passwords, personal data, and file contents.

# Syntax

```dart
debugPrint('GET /v1/items status=$status elapsedMs=$elapsed requestId=$id');
```

# Complete Example 1

This runnable diagnostic screen calls a demo endpoint and displays timing/status. Add `http`.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const ApiDebugApp());

class ApiDebugApp extends StatelessWidget {
  const ApiDebugApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: ApiDebugPage());
}

class ApiDebugPage extends StatefulWidget {
  const ApiDebugPage({super.key});

  @override
  State<ApiDebugPage> createState() => _ApiDebugPageState();
}

class _ApiDebugPageState extends State<ApiDebugPage> {
  String _result = 'Ready';
  bool _loading = false;

  Future<void> _probe() async {
    final watch = Stopwatch()..start();
    setState(() => _loading = true);
    try {
      final response = await http.get(
        Uri.https('jsonplaceholder.typicode.com', '/posts/1'),
      );
      watch.stop();
      if (!mounted) return;
      setState(() =>
          _result = 'status=${response.statusCode}, elapsed=${watch.elapsedMilliseconds}ms');
    } catch (error) {
      watch.stop();
      if (!mounted) return;
      setState(() => _result = 'Transport/parser failure after ${watch.elapsedMilliseconds}ms');
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('API diagnostics')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_result, textAlign: TextAlign.center),
              FilledButton(
                onPressed: _loading ? null : _probe,
                child: const Text('Probe endpoint'),
              ),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `Stopwatch` measures elapsed time without exposing sensitive payload.
- `_probe` starts loading and sends a known GET request.
- On a response, it reports HTTP status and elapsed milliseconds.
- Exceptions are categorized as a request-stage failure; production code should distinguish timeout, DNS, TLS, and parser errors.
- `finally` always clears loading.
- The button is disabled to avoid overlapping probes.

### Widget tree

```text
MaterialApp → ApiDebugPage → Scaffold → Center → Column(result, button)
```

### Execution flow

```text
Start stopwatch -> send request -> response/exception -> record timing
                 -> update diagnostic UI -> stop loading
```

### ASCII UI preview

```text
+---------------------------+
|     API diagnostics       |
+---------------------------+
| status=200, elapsed=...ms |
| [     Probe endpoint   ]  |
+---------------------------+
```

### Example explanation

This probe differentiates an HTTP response from a thrown request failure and measures duration. It is not a full tracing system; include request IDs and sanitized logs in production diagnostics.

If errors are swallowed with no status/timing, diagnosis is difficult. If request headers are dumped, bearer tokens may leak.

**Common mistakes:** Do not attach production API logs containing personal data to public issue trackers.

# Complete Example 2

Use a mock client to test status and parsing:

```dart
final client = MockClient((request) async => http.Response('{"id":1}', 200));
```

The test can validate URI, method, headers, and model mapping without a live network.

# Complete Example 3

Use a correlation ID end-to-end:

```text
Client request ID -> response header/log -> server trace lookup
```

This helps connect client and backend diagnostics.

# Advanced Example

Create a failure matrix covering DNS, TLS, timeout, 401, 429, 500, invalid JSON, wrong field type, and cancellation. For each case specify expected user state, diagnostic metadata, and retry policy.

# Under the Hood

Exceptions can originate from the socket layer, TLS handshake, HTTP client, JSON parser, model factory, or widget lifecycle. Status codes only exist when a normal HTTP response was received.

# Performance Notes

- Collect durations and outcome categories.
- Avoid verbose body logging.
- Use sampling for high-volume telemetry.
- Keep diagnostic tooling disabled or sanitized in release builds.

# Best Practices

- Trace one request from UI to server and back.
- Record request IDs and timing.
- Redact secrets and private payloads.
- Use mock clients and fixtures.
- Reproduce on target platform/build mode.

# Common Mistakes

- Assuming all failures are HTTP statuses.
- Logging tokens/bodies.
- Testing only on a warm app.
- Changing multiple layers at once without evidence.
- Debugging against a different API environment.

# Debugging

1. Verify method and final URI.
2. Verify network/DNS/TLS.
3. Inspect status/headers.
4. Validate body format and model mapping.
5. Confirm state update and widget subscription.

# Interview Questions

1. What is the difference between transport error and HTTP error? No response versus received status.
2. What should a safe API log include? Sanitized route, method, status, timing, request ID.
3. Why mock HTTP? Deterministic tests without network dependence.
4. What does a request ID provide? Correlation between client and server traces.

# Try It Yourself

1. Add timeout categorization.
2. Write a mock-client test for 404.
3. Add request IDs.
4. Remove sensitive fields from diagnostic output.

# Challenge Project

Build a debug-only API inspector that reports timing, status, request ID, and redacted headers, with filters for transport and parsing errors.

# Quick Revision Sheet

- Trace request through every layer.
- Transport errors have no HTTP status.
- Log method/path/status/time/request ID.
- Redact credentials and user data.
- Use mock clients and failure matrices.

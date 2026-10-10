# Introduction

A timeout limits how long an app waits for an operation. Network calls can stall because of connectivity, server load, or transport issues; the UI should not wait indefinitely.

# Why this concept exists

Without a timeout, a spinner may remain forever and resources may remain occupied. A timeout lets the app offer retry or fallback behavior.

# Real-world analogy

A phone call has a ring limit. If nobody answers, you can call back or try another route instead of holding the line forever.

# Internal Working

In Dart, `Future.timeout` completes with a timeout error if the source future does not complete before the duration. It does not necessarily cancel the underlying socket/request. Use the client API's cancellation support when true cancellation is needed.

# Syntax

```dart
final response = await client.get(uri).timeout(
  const Duration(seconds: 10),
);
```

# Complete Example 1

This screen calls a public endpoint with an 8-second timeout and reports the result. Add `http`.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const TimeoutApp());

class TimeoutApp extends StatelessWidget {
  const TimeoutApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: TimeoutPage());
}

class TimeoutPage extends StatefulWidget {
  const TimeoutPage({super.key});

  @override
  State<TimeoutPage> createState() => _TimeoutPageState();
}

class _TimeoutPageState extends State<TimeoutPage> {
  String _message = 'Ready';
  bool _loading = false;

  Future<void> _request() async {
    setState(() => _loading = true);
    try {
      final response = await http
          .get(Uri.https('jsonplaceholder.typicode.com', '/posts/1'))
          .timeout(const Duration(seconds: 8));
      if (!mounted) return;
      setState(() => _message = 'Response status ${response.statusCode}');
    } on Exception catch (_) {
      if (!mounted) return;
      setState(() => _message = 'Request timed out or failed. Retry when ready.');
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Request timeout')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_message),
              FilledButton(
                onPressed: _loading ? null : _request,
                child: const Text('Request'),
              ),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `_request` begins loading before awaiting the call.
- `.timeout` bounds the wait to eight seconds.
- Success displays the returned status.
- Failure or timeout produces a recoverable message.
- `finally` clears loading for every completion path.
- The button disables during the request.

### Widget tree

```text
MaterialApp → TimeoutPage → Scaffold → Center → Column(status, button)
```

### Execution flow

```text
Request starts -> response before deadline? -> show status
                 timeout/error -> show retry message
                 finally -> stop spinner
```

### ASCII UI preview

```text
+---------------------------+
|     Request timeout       |
+---------------------------+
| Request timed out or      |
| failed. Retry when ready. |
| [        Request        ] |
+---------------------------+
```

### Example explanation

Timeout is an error path, not an HTTP status. The underlying request might still be running depending on client cancellation support, so a retry can overlap. For write operations, that creates duplicate risk.

If the timeout is removed, the UI can wait too long. If `finally` is removed, the loading button can remain disabled.

**Common mistakes:** Do not choose one short timeout for every upload, download, and API call.

# Complete Example 2

Configure connection and response timeouts separately if the client supports them:

```text
connection timeout: time to establish transport
response timeout: time waiting for response bytes
```

Package-specific behavior differs; inspect API documentation.

# Complete Example 3

Retry safe GET with bounded backoff:

```text
attempt 1 -> wait 300ms -> attempt 2 -> wait 700ms -> final error
```

Use jitter and respect rate-limit guidance to avoid synchronized retry bursts.

# Advanced Example

Choose deadlines based on user experience and payload type. Propagate a total request deadline across retries rather than resetting the full timeout indefinitely. Cancel obsolete search requests where possible.

# Under the Hood

`Future.timeout` races the original future against a timer. The timeout future can complete while the original operation continues unless the transport is explicitly canceled.

# Performance Notes

- Limit concurrent hung requests.
- Cancel obsolete work.
- Use separate budgets for small JSON and large transfers.
- Prevent retry storms.

# Best Practices

- Define appropriate timeouts.
- Distinguish timeout from HTTP error.
- Provide retry UX.
- Make write retries idempotent.
- Use cancellation when supported.

# Common Mistakes

- Assuming `Future.timeout` cancels the socket.
- Retrying POST after timeout without idempotency.
- Too-short timeout for large files.
- Infinite retries.

# Debugging

- Log elapsed time and operation type.
- Check DNS, TLS, connection, and response phase.
- Verify timeout applies to the intended future.
- Test with controlled slow server/mock.

# Interview Questions

1. Does `Future.timeout` cancel the source future? Not necessarily.
2. Why are timeouts important? Bound resource and UX waiting.
3. How retry after timeout safely? Only if idempotent/replayable or protected by idempotency key.
4. Should upload and GET share timeout? Not necessarily; payload size and UX differ.

# Try It Yourself

1. Distinguish timeout from 500.
2. Add retry with a maximum attempt count.
3. Test a canceled obsolete search.
4. Define transfer-specific deadlines.

# Challenge Project

Build a configurable API client with operation-specific deadlines, cancellation for search, and bounded retries only for eligible requests.

# Quick Revision Sheet

- Timeout bounds waiting.
- It may not cancel the underlying request.
- Separate transport timeout from HTTP status.
- Retry only safe/replayable operations.
- Always clear loading state.

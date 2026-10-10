# Introduction

Connectivity monitoring reports whether a device appears connected to a network interface such as Wi-Fi or cellular. It does not prove that the internet or a particular API is reachable.

# Why this concept exists

Apps can adjust UX when offline, but connectivity state is only a hint. The actual request result is the reliable signal for a server operation.

# Real-world analogy

Seeing a road open on a map does not prove the destination's door is open. Network availability does not prove DNS, routing, server health, or authentication is working.

# Internal Working

Connectivity plugins observe platform network state. A network can be connected without internet access, behind a captive portal, or unable to reach one service. Always handle request exceptions/statuses even when the connectivity plugin says online.

# Syntax

```dart
final result = await Connectivity().checkConnectivity();
```

The `connectivity_plus` API has changed across versions; inspect current return types and stream behavior when adding the dependency.

# Complete Example 1

This dependency-free app demonstrates a user-facing retry that trusts the actual HTTP request, not a connectivity badge. Add `http`.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const ConnectivityApp());

class ConnectivityApp extends StatelessWidget {
  const ConnectivityApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: ConnectivityPage());
}

class ConnectivityPage extends StatefulWidget {
  const ConnectivityPage({super.key});

  @override
  State<ConnectivityPage> createState() => _ConnectivityPageState();
}

class _ConnectivityPageState extends State<ConnectivityPage> {
  String _message = 'Request not tested';

  Future<void> _test() async {
    setState(() => _message = 'Checking service...');
    try {
      final response = await http
          .get(Uri.https('jsonplaceholder.typicode.com', '/posts/1'))
          .timeout(const Duration(seconds: 6));
      if (!mounted) return;
      setState(() => _message = 'Service responded: ${response.statusCode}');
    } catch (error) {
      if (!mounted) return;
      setState(() => _message = 'Service could not be reached');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Connectivity is a hint')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_message),
              FilledButton(onPressed: _test, child: const Text('Test service')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `_test` attempts the real service call and applies a timeout.
- Any HTTP response status proves a server response arrived, though it may indicate an API error.
- A thrown exception indicates no normal response.
- The UI reports service reachability rather than assuming Wi-Fi state equals internet.

### Widget tree

```text
MaterialApp → ConnectivityPage → Scaffold → Center → Column
                                                  ├→ result
                                                  └→ test button
```

### Execution flow

```text
Connectivity indicator (hint) != API result
Request -> server response status OR transport failure -> update UI
```

### ASCII UI preview

```text
+---------------------------+
| Connectivity is a hint    |
+---------------------------+
| Service responded: 200    |
| [       Test service    ] |
+---------------------------+
```

### Example explanation

This example intentionally does not gate the request on a network-interface plugin. Attempting a request handles captive portals, DNS failure, and service-specific outages more accurately.

If the app refuses to request whenever a connectivity plugin reports “none,” it may miss a working route or delay a recovery. If connectivity reports Wi-Fi, the server can still be unavailable.

**Common mistakes:** Never use connectivity state as the sole source of truth for request success.

# Complete Example 2

A connectivity listener can update an offline banner:

```dart
connectivityStream.listen((status) {
  // Update a hint/banner, but keep handling request failures.
});
```

Cancel the subscription when its owner is disposed.

# Complete Example 3

Queue a local write regardless of network state, then schedule sync opportunistically. This is stronger offline UX than blocking user action behind a connectivity check.

# Advanced Example

Use connectivity transitions to trigger a sync attempt, with backoff and server validation. Avoid performing every queued request at once after reconnecting. Respect battery, metered-network, and user data policies.

# Under the Hood

Platform connectivity APIs report interface status and may emit changes. They do not execute DNS resolution or an HTTP health check to every service. Only the request path reveals actual endpoint availability.

# Performance Notes

- Avoid polling connectivity aggressively.
- Coalesce reconnect-triggered sync.
- Do not fire large transfers automatically on metered networks without policy.

# Best Practices

- Treat connectivity as advisory.
- Keep normal request error handling.
- Support offline local reads/writes where required.
- Cancel subscriptions.
- Provide retry and cached content.

# Common Mistakes

- Equating Wi-Fi with internet.
- Equating internet with API availability.
- Blocking a user action unnecessarily.
- Launching duplicate sync on repeated network events.

# Debugging

- Compare interface state with actual endpoint request.
- Test captive portal, VPN, airplane mode, and DNS failure.
- Inspect request timeout/status.
- Check subscription cleanup and duplicate triggers.

# Interview Questions

1. Does connectivity prove API reachability? No.
2. What does a connectivity plugin report? Network interface status.
3. What is authoritative for an operation? Its actual request result.
4. How use connectivity events? As a hint to attempt sync/retry, not guarantee success.

# Try It Yourself

1. Add an offline banner using a connectivity plugin.
2. Keep request handling independent of the banner.
3. Simulate Wi-Fi with no internet.
4. Coalesce repeated reconnect events.

# Challenge Project

Build a network-aware notes screen that works from local data offline, shows an advisory banner, and syncs safely after reconnect.

# Quick Revision Sheet

- Connectivity plugin reports interface, not service health.
- Actual request determines endpoint reachability.
- Keep offline UX and errors independent.
- Use network changes as sync hints.
- Debounce reconnect-driven work.

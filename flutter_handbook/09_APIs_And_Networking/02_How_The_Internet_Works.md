# Introduction

The internet is a network of connected devices that exchange data using agreed protocols. A Flutter app usually speaks HTTP or HTTPS to a server identified by a domain name.

# Why this concept exists

Understanding the path from a URI to a response makes network failures easier to diagnose. “The API is down” may actually be a DNS, TLS, routing, timeout, authentication, or application error.

# Real-world analogy

Sending a letter requires an address, a route through postal hubs, a recipient, and a reply path. Internet communication similarly resolves names, routes packets, establishes secure transport, and exchanges protocol messages.

# Internal Working

Typical HTTPS request path:

```text
URI -> DNS resolves hostname -> connection established
    -> TLS negotiates encryption/certificates
    -> HTTP request sent -> server handles it
    <- HTTP response bytes <- connection reused/closed
```

TCP provides ordered reliable byte delivery; TLS adds encryption and server authentication; HTTP defines request/response semantics. HTTP/2 and HTTP/3 can change transport details, but application code still works with request and response abstractions.

# Syntax

```dart
final uri = Uri.https(
  'api.example.com',
  '/v1/weather',
  {'city': 'Auckland'},
);
```

Using `Uri.https` avoids manually concatenating and incorrectly escaping URI parts.

# Complete Example 1

This app sends an HTTPS request and displays the response status. Add `http` to dependencies; it uses a public test endpoint.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const InternetPathApp());

class InternetPathApp extends StatelessWidget {
  const InternetPathApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: NetworkPage());
}

class NetworkPage extends StatefulWidget {
  const NetworkPage({super.key});

  @override
  State<NetworkPage> createState() => _NetworkPageState();
}

class _NetworkPageState extends State<NetworkPage> {
  String _status = 'Not requested';
  bool _loading = false;

  Future<void> _checkServer() async {
    setState(() {
      _loading = true;
      _status = 'Connecting...';
    });
    try {
      final uri = Uri.https('jsonplaceholder.typicode.com', '/posts/1');
      final response = await http.get(uri).timeout(const Duration(seconds: 8));
      if (!mounted) return;
      setState(() => _status = 'HTTP ${response.statusCode}');
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Network request failed');
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Internet request')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_status),
              const SizedBox(height: 12),
              FilledButton(
                onPressed: _loading ? null : _checkServer,
                child: const Text('Send HTTPS request'),
              ),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- The UI and `http` imports provide a button and an HTTP client function.
- `_status` communicates the current phase/result; `_loading` prevents duplicate taps.
- `_checkServer` updates state before awaiting the request.
- `Uri.https` names host and path separately.
- `timeout` bounds how long the operation waits.
- The response status is displayed; a transport exception is reported as a network failure.
- `finally` clears loading whether the request succeeds or throws.
- `mounted` protects every asynchronous state update.
- The button disables while loading.

### Widget tree

```text
MaterialApp -> NetworkPage -> Scaffold -> Center -> Column
                                                   ├-> status
                                                   └-> request button
```

### Execution flow

```text
Tap -> Dart HTTP client -> DNS -> TCP/TLS -> HTTP server
     <- response status/body <- network stack
     -> state update -> Flutter rebuild
```

### ASCII UI preview

```text
+---------------------------+
|     Internet request      |
+---------------------------+
| HTTP 200                  |
| [ Send HTTPS request ]    |
+---------------------------+
```

### Example explanation

This demonstrates that a request is asynchronous and can fail before an HTTP status is received. A 404 or 500 is still an HTTP response; a DNS/TLS/timeout exception is a transport-level failure.

If the `finally` block is removed, the button can remain disabled after failure. If the timeout is removed, some network calls may wait far longer than the UX allows. The catch avoids exposing raw internals to users; production logging should capture sanitized diagnostics.

**Common mistakes:** Do not interpret “connected to Wi-Fi” as “server reachable”; DNS and server health are separate.

**Professional use case:** Connectivity diagnostics and structured network error categorization.

# Complete Example 2

DNS maps a hostname to network addresses. Application code should use a hostname in its URI and let the operating system resolver handle address changes:

```dart
final uri = Uri.https('api.example.com', '/health');
```

Hard-coding an IP can break when infrastructure changes and can interfere with TLS certificate hostname validation.

# Complete Example 3

HTTPS validates the server's certificate as part of TLS negotiation. Use standard certificate validation; do not disable checks to “fix” a development certificate error.

# Advanced Example

Differentiate failures: name resolution, socket connection, TLS negotiation, timeout, non-success HTTP status, and invalid payload. Each points to a different layer and may have a different retry policy.

# Under the Hood

The Dart HTTP package delegates socket operations to the platform/runtime networking stack. DNS lookup, connection reuse, TLS certificate checks, and packet routing occur below JSON parsing and widget rendering.

# Performance Notes

- Reuse connections through a shared client.
- Set appropriate timeouts.
- Request only required payload data.
- Avoid repeated calls on rebuild.
- Compress and paginate large responses where APIs support them.

# Best Practices

- Use HTTPS.
- Use DNS hostnames and URI builders.
- Distinguish transport failures from HTTP errors.
- Avoid disabling certificate validation.
- Provide offline-friendly UI and retry controls.

# Common Mistakes

- Treating a Wi-Fi indicator as proof of API availability.
- Hard-coding server IPs.
- Ignoring TLS certificate failures.
- Retrying every failure identically.
- Performing synchronous network work.

# Debugging

- Check hostname resolution, certificate validity, endpoint, and device date.
- Compare on simulator and physical device.
- Inspect proxy/VPN/firewall configuration.
- Test timeout separately from server status errors.

# Interview Questions

1. What does DNS do? Resolves hostnames to network addresses.
2. What does TLS add? Encrypted transport and certificate-based server authentication.
3. Is HTTP 500 a network failure? It is an HTTP response indicating a server-side error.
4. Why are connection reuse and HTTP/2 useful? They can reduce connection setup overhead and improve multiplexing.

# Try It Yourself

1. Test a valid endpoint and an invalid hostname.
2. Add separate timeout and HTTP error messages.
3. Try a request with airplane mode enabled.
4. Draw the DNS/TLS/HTTP path.

# Challenge Project

Build a network diagnostics screen that classifies DNS, timeout, HTTP, and parsing errors without exposing sensitive details.

# Quick Revision Sheet

- DNS resolves names.
- TCP transports ordered bytes.
- TLS protects/authenticates HTTPS.
- HTTP carries method, headers, and body.
- Transport failure is distinct from HTTP error status.

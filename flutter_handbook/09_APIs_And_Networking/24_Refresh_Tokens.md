# Introduction

An access token is often short-lived. A refresh token can be exchanged with an authorization server for a new access token without asking the user to sign in again.

# Why this concept exists

Short-lived access tokens reduce the impact window if one is exposed, while refresh tokens support longer sessions with controlled renewal.

# Real-world analogy

An access pass expires quickly, but a secure renewal credential can obtain a new pass after the desk verifies that the account remains valid.

# Internal Working

When a protected request returns an authentication failure, the client may attempt a refresh according to the provider's protocol. Refresh tokens are sent only to the authorization endpoint, stored securely, and may rotate. Concurrent failures should share one refresh operation. If refresh fails, clear session state and require sign-in.

# Syntax

```text
401 -> refresh credential -> new access token -> retry eligible request once
```

# Complete Example 1

This runnable demonstration models refresh coordination with a fake service. It does not contact an auth server or persist credentials.

```dart
import 'package:flutter/material.dart';

class FakeSessionService {
  int refreshCount = 0;

  Future<String> refresh(String refreshToken) async {
    await Future<void>.delayed(const Duration(milliseconds: 400));
    if (refreshToken.isEmpty) throw StateError('Refresh credential missing');
    refreshCount++;
    return 'fake-access-${refreshCount}';
  }
}

void main() => runApp(const RefreshTokenApp());

class RefreshTokenApp extends StatelessWidget {
  const RefreshTokenApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: RefreshTokenPage());
}

class RefreshTokenPage extends StatefulWidget {
  const RefreshTokenPage({super.key});

  @override
  State<RefreshTokenPage> createState() => _RefreshTokenPageState();
}

class _RefreshTokenPageState extends State<RefreshTokenPage> {
  final _service = FakeSessionService();
  String _status = 'Access token expired (simulated)';
  bool _loading = false;

  Future<void> _refresh() async {
    setState(() => _loading = true);
    try {
      final access = await _service.refresh('fake-refresh-token');
      if (!mounted) return;
      setState(() => _status = 'New demo access token: $access');
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Session expired; sign in again');
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Token refresh')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_status, textAlign: TextAlign.center),
              FilledButton(
                onPressed: _loading ? null : _refresh,
                child: const Text('Refresh session'),
              ),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `FakeSessionService` simulates latency and issues a fake access token.
- `_refresh` starts a loading state, awaits the fake refresh, and handles failure.
- `finally` always resets loading.
- The button is disabled during the operation, reducing repeated calls in this single-widget demo.
- The UI clearly labels the token as fake.

### Widget tree

```text
MaterialApp → RefreshTokenPage → Scaffold → Center → Column(status, button)
```

### Execution flow

```text
Protected request gets 401 (simulated)
 -> refresh service -> new access token
 -> retry eligible request once
 -> refresh failure clears session
```

### ASCII UI preview

```text
+---------------------------+
|       Token refresh       |
+---------------------------+
| New demo access token     |
| [ Refresh session ]       |
+---------------------------+
```

### Example explanation

This sample demonstrates asynchronous state and failure handling only. Production refresh uses a provider-defined endpoint, secure storage, response validation, and token rotation. It must not display or log actual tokens.

If loading disable is removed, repeated taps can create concurrent refreshes. If refresh failure does not clear session state, protected requests can loop forever.

**Common mistakes:** Do not send a refresh token to ordinary resource endpoints.

# Complete Example 2

Single-flight refresh coordination shares one Future:

```dart
Future<String>? _refreshing;
```

If many requests receive 401 together, await the existing refresh rather than sending many refresh calls.

# Complete Example 3

Retry the original request only once after refresh and only if replay is safe or the request body can be replayed reliably.

# Advanced Example

Handle rotating refresh tokens atomically: store the new token before discarding the old according to provider contract. Detect reuse, revoke compromised sessions where supported, and coordinate refresh across isolates/processes if the app architecture permits concurrent access.

# Under the Hood

OAuth token endpoints authenticate the client/user context and issue credentials. Refresh tokens may be opaque or structured; clients should treat them as secrets and follow provider policy. A 401 can mean more than expiry, so refresh only for documented auth failures.

# Performance Notes

- Refresh before expiry when safe, or on demand.
- Coalesce concurrent refreshes.
- Bound retries and use timeouts.
- Avoid refreshing on every widget build.

# Best Practices

- Securely store refresh token.
- Follow rotation/revocation policy.
- Use single-flight concurrency.
- Retry at most once and safely.
- Clear session on definitive failure.

# Common Mistakes

- Infinite 401-refresh-retry loops.
- Concurrent refresh races.
- Persisting refresh token in preferences.
- Retrying non-repeatable request bodies.
- Logging credentials.

# Debugging

- Log refresh attempt IDs/status only.
- Check token rotation behavior and expiry.
- Test several simultaneous 401 responses.
- Simulate timeout after token rotation.
- Verify sign-out clears in-memory and secure credentials.

# Interview Questions

1. Why separate access and refresh tokens? Different lifetimes and risk exposure.
2. Where store refresh token? Secure storage.
3. What is single-flight refresh? Coalescing concurrent refresh requests into one operation.
4. Why retry only once? Prevent infinite loops.
5. What is token rotation? Issuing a new refresh token and invalidating prior one.

# Try It Yourself

1. Implement a shared refresh Future in a fake client.
2. Simulate refresh failure and sign-out.
3. Test parallel protected requests.
4. Define which request bodies are replayable.

# Challenge Project

Build a mock API client with 401 handling, secure-token interface, single-flight refresh, one retry, and tests for refresh rotation/failure.

# Quick Revision Sheet

- Access tokens expire; refresh tokens renew under policy.
- Store refresh credentials securely.
- Coalesce concurrent refresh requests.
- Retry eligible request once.
- Handle rotation and definitive failure.

# Introduction

Login-state persistence remembers enough information for an app to restore or validate a user's session after restart. It does not mean storing a boolean and trusting it forever.

# Why this concept exists

Users should not have to sign in on every app launch when a valid session can be securely restored. At the same time, sessions expire, tokens can be revoked, and local devices can be shared.

# Real-world analogy

A secure access badge can make returning to a building convenient, but the building still checks whether the badge is valid and has not been revoked. A note saying “I am authorized” is not the badge.

# Internal Working

Authentication typically returns a short-lived access token and possibly a refresh credential. Store sensitive credentials using platform-backed secure storage, not plain preferences. On startup, read the credential, contact the trusted service to refresh or validate the session, then expose authenticated state.

Clear credentials on sign-out and handle invalid/expired tokens. A local `isLoggedIn` flag can be a UI hint but must never authorize protected server operations.

# Syntax

```dart
await secureStorage.write(key: 'refreshToken', value: token);
final token = await secureStorage.read(key: 'refreshToken');
```

The secure-storage object comes from a package such as `flutter_secure_storage`; add its dependency and platform configuration. Never print tokens in logs.

# Complete Example 1

This is a runnable UI-flow example using `flutter_secure_storage`. It demonstrates a saved session marker and explicit sign-out. It is not a complete authentication system; production apps must validate credentials with their backend. Add the package and complete its platform setup first.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

void main() => runApp(const SessionApp());

class SessionApp extends StatelessWidget {
  const SessionApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: SessionPage());
}

class SessionPage extends StatefulWidget {
  const SessionPage({super.key});

  @override
  State<SessionPage> createState() => _SessionPageState();
}

class _SessionPageState extends State<SessionPage> {
  static const _storage = FlutterSecureStorage();
  bool _loading = true;
  bool _hasSession = false;

  @override
  void initState() {
    super.initState();
    _restore();
  }

  Future<void> _restore() async {
    final token = await _storage.read(key: 'refreshToken');
    // A real app validates or refreshes this credential with its backend.
    if (!mounted) return;
    setState(() {
      _hasSession = token != null;
      _loading = false;
    });
  }

  Future<void> _demoSignIn() async {
    await _storage.write(key: 'refreshToken', value: 'demo-only-token');
    if (!mounted) return;
    setState(() => _hasSession = true);
  }

  Future<void> _signOut() async {
    await _storage.delete(key: 'refreshToken');
    if (!mounted) return;
    setState(() => _hasSession = false);
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Session restoration')),
        body: Center(
          child: _loading
              ? const CircularProgressIndicator()
              : Column(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    Text(_hasSession ? 'Session marker found' : 'Signed out'),
                    FilledButton(
                      onPressed: _hasSession ? _signOut : _demoSignIn,
                      child: Text(_hasSession ? 'Sign out' : 'Demo sign in'),
                    ),
                  ],
                ),
        ),
      );
}
```

### Line-by-line explanation

- The secure-storage package exposes platform-backed credential storage.
- `_storage` is a constant client; `_loading` and `_hasSession` drive display.
- `_restore` reads a token asynchronously. This demonstration checks presence only; production code must validate or refresh it remotely.
- `_demoSignIn` writes a clearly labeled fake token for the UI demonstration only.
- `_signOut` deletes the token and updates state after the delete completes.
- The UI displays loading, session-marker, or signed-out state.

### Widget tree

```text
MaterialApp → SessionPage → Scaffold → Center
                                      ├→ loading indicator
                                      └→ status + action button
```

### Execution flow

```text
Startup → secure read → backend validation required in production
Demo sign-in → secure write → show demonstration state
Sign-out → delete credential → clear authenticated UI state
```

### ASCII phone preview

```text
+---------------------------+
| Session restoration       |
+---------------------------+
| Session marker found      |
| [       Sign out       ]  |
+---------------------------+
```

### Example explanation

The secure store protects small credentials better than a plain preference store, but it does not establish that the server still accepts a session. The demonstration token is not a real credential and must not be used for authentication.

If the credential read is removed, restart restoration is impossible. If deletion on sign-out is removed, the next launch may restore the old session. If backend validation is omitted in production, revoked or expired sessions may be treated as valid locally.

**Common mistakes:** Never hard-code a token or log it. Use OAuth/OIDC or the service's approved authentication flow and follow secure token lifecycle guidance.

**Professional use case:** Restoring a user session through secure token storage and server-side refresh validation.

# Complete Example 2

Keep a non-secret display hint separately:

```dart
await prefs.setString('lastAccountLabel', 'Personal');
```

This can help select a sign-in screen, but must not grant access. Avoid storing unnecessary identifying information.

# Complete Example 3

On token rejection, clear credentials and return to signed-out state:

```dart
if (response.statusCode == 401) {
  await secureStorage.delete(key: 'refreshToken');
  sessionController.signOut();
}
```

The session controller should clear account-scoped in-memory data as well as update navigation state.

# Advanced Example

Model session startup as `unknown/loading`, `authenticated`, `unauthenticated`, or `error`. Read a credential, refresh it securely, handle rotation atomically as supported, then fetch account state. On sign-out, revoke server credentials where supported and clear local secrets and sensitive caches.

# Under the Hood

Secure-storage plugins use operating-system credential facilities (for example Keychain or encrypted Android storage, depending on configuration). Platform guarantees differ. Data may be unavailable after device restore, biometric changes, or OS policy changes; handle read errors and migration requirements.

# Performance Notes

- Keep secrets small.
- Do not block the first frame while showing unrelated UI; use a deliberate auth gate.
- Do not repeatedly read tokens from secure storage for every API call; use a controlled in-memory session provider with refresh policy.
- Avoid expensive repeated authentication checks.

# Best Practices

- Use a trusted auth protocol/service.
- Store refresh credentials in platform secure storage.
- Validate/refresh on startup and handle expiry.
- Clear/revoke credentials on sign-out.
- Never log or embed credentials.
- Treat local flags as hints, not authorization.

# Common Mistakes

- Persisting `isLoggedIn=true` in preferences as the only check.
- Saving tokens in plain text preferences.
- Hard-coding demo credentials.
- Logging sensitive credentials.
- Ignoring token expiry, revocation, rotation, and sign-out cleanup.

# Debugging

- Inspect error categories without revealing token content.
- Test expired, missing, and rejected credentials.
- Verify sign-out removes secure values and account memory.
- Test platform setup on each target platform.
- Ensure startup errors do not trap the user on a blank screen.

# Interview Questions

1. Where should a refresh token be stored? Platform-backed secure storage.
2. Is a local login flag proof of authorization? No.
3. What should happen at startup? Read credential, validate/refresh with the trusted service, and resolve session state.
4. What does sign-out require? Clear local state and credentials, and revoke remotely when supported.
5. Should tokens be logged? Never.

# Try It Yourself

1. Replace the demo marker logic with a mocked validation service.
2. Add an explicit session-error state and retry action.
3. Clear account-specific cached records on sign-out.
4. Write a checklist for token expiration and refresh.

# Challenge Project

Build an authentication session coordinator with loading, authenticated, unauthenticated, and recoverable error states. Use a fake service in the sample, secure storage for the token boundary, and no hard-coded real credentials.

# Quick Revision Sheet

- Login persistence means securely restoring and validating a session.
- Secure storage is for small secrets; preferences are not.
- Validate/refresh credentials with the backend.
- Clear credentials and account state on sign-out.
- Never trust a local boolean or log a token.

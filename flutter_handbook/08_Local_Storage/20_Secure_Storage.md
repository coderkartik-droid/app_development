# Introduction

Secure storage is a place for small secrets such as refresh tokens or cryptographic keys, backed by operating-system security facilities. In Flutter, packages bridge Dart code to facilities such as iOS Keychain and Android secure storage mechanisms.

# Why this concept exists

Ordinary preferences and app files are not designed as secret vaults. Credentials need stronger protection and a lifecycle that includes expiry, rotation, revocation, and deletion.

# Real-world analogy

Secure storage is a locked key cabinet, not a safe deposit box that makes every risk disappear. The operating system controls access, but the app must still protect the key and handle compromised devices.

# Internal Working

Secure-storage plugins call native APIs. Exact encryption, backup, biometric, and migration behavior depends on platform and plugin configuration. Reads and writes are asynchronous and can fail due to OS policy or device state.

Use secure storage for small credentials. Store large data in a database with a deliberate encryption/key strategy, not as a huge secure-storage value.

# Syntax

```dart
const storage = FlutterSecureStorage();
await storage.write(key: 'refreshToken', value: token);
final token = await storage.read(key: 'refreshToken');
```

Add the package dependency and complete platform-specific setup from its current documentation.

# Complete Example 1

This small app stores a fake API key and offers deletion. It is a UI demonstration only; never embed real secrets in an app binary. Add and configure `flutter_secure_storage`.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

void main() => runApp(const SecureStorageApp());

class SecureStorageApp extends StatelessWidget {
  const SecureStorageApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: SecureStoragePage());
}

class SecureStoragePage extends StatefulWidget {
  const SecureStoragePage({super.key});

  @override
  State<SecureStoragePage> createState() => _SecureStoragePageState();
}

class _SecureStoragePageState extends State<SecureStoragePage> {
  static const _storage = FlutterSecureStorage();
  final _input = TextEditingController();
  bool _exists = false;

  Future<void> _saveDemoValue() async {
    final value = _input.text.trim();
    if (value.isEmpty) return;
    await _storage.write(key: 'demoSecret', value: value);
    if (!mounted) return;
    setState(() {
      _exists = true;
      _input.clear();
    });
  }

  Future<void> _delete() async {
    await _storage.delete(key: 'demoSecret');
    if (!mounted) return;
    setState(() => _exists = false);
  }

  @override
  void dispose() {
    _input.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Secure storage demo')),
        body: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            children: [
              Text(_exists ? 'A value is stored (hidden)' : 'No value stored'),
              TextField(
                controller: _input,
                obscureText: true,
                decoration: const InputDecoration(labelText: 'Demo value'),
              ),
              FilledButton(
                onPressed: _saveDemoValue,
                child: const Text('Store'),
              ),
              OutlinedButton(
                onPressed: _exists ? _delete : null,
                child: const Text('Delete'),
              ),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- The secure-storage package is imported alongside Material UI.
- `_storage` accesses the plugin; `_input` collects a demonstration value; `_exists` reveals presence only.
- `_saveDemoValue` rejects empty input and writes the value under a fixed key.
- The app deliberately never reads or prints the secret.
- `_delete` removes the key.
- `mounted` guards state updates after asynchronous operations.
- `obscureText` visually hides typed input.
- Delete is disabled if no value is believed to exist.
- `dispose` releases the text controller.

### Widget tree

```text
MaterialApp → SecureStoragePage → Scaffold → Padding → Column
                                                    ├→ status
                                                    ├→ secret TextField
                                                    ├→ Store
                                                    └→ Delete
```

### Execution flow

```text
Enter demo value → secure write → show stored marker
Delete → secure delete → clear marker
```

### ASCII phone preview

```text
+---------------------------+
| Secure storage demo       |
+---------------------------+
| A value is stored (hidden)|
| Demo value                |
| [ ********             ]  |
| [         Store        ]  |
| [         Delete       ]  |
+---------------------------+
```

### Example explanation

The value is never displayed after storage, demonstrating data minimization. Secure storage protects local credential material better than plain preferences, but embedding a secret in a distributed application cannot make it confidential from a determined reverse engineer.

If the secure-storage write is replaced with a preference write, the data protection changes. If the controller is not disposed, UI resources leak; if a real key is logged or hard-coded, platform storage does not undo that exposure.

**Common mistakes:** A client-side API key intended to be secret cannot reliably be hidden in a public mobile binary. Use server-side credentials or appropriately restricted public keys.

**Professional use case:** Store a refresh token returned after user authentication.

# Complete Example 2

Read a credential only when session restoration needs it:

```dart
final refreshToken = await storage.read(key: 'refreshToken');
if (refreshToken == null) {
  return SessionState.signedOut;
}
```

The next step must validate or refresh with the trusted service. Do not treat presence as validity.

# Complete Example 3

Delete a token on sign-out:

```dart
await storage.delete(key: 'refreshToken');
```

Clear associated in-memory account state as well. Consider server-side revocation if the authentication system supports it.

# Advanced Example

Plan secure-storage migration when changing plugin versions, encryption options, app identifiers, or platform configuration. Test device restore, biometric changes, and key-unavailable behavior. Never silently replace a failed secure read with a new authenticated state.

# Under the Hood

The plugin bridges Dart calls to native credential APIs. Storage protection and accessibility classes are platform-specific. Android backup behavior and iOS Keychain accessibility need careful review in the plugin documentation.

# Performance Notes

- Store small secrets only.
- Do not call secure storage repeatedly during each widget build or API request.
- Keep an in-memory session object after controlled startup validation.
- Avoid blocking unrelated startup UI on secure reads.

# Best Practices

- Use platform-backed secure storage for small secrets.
- Never log credentials.
- Validate tokens with the backend.
- Delete/revoke on sign-out.
- Use least privilege and token expiration.
- Review platform backup/accessibility settings.

# Common Mistakes

- Saving tokens in preferences or source code.
- Assuming encrypted storage solves all threat models.
- Treating token presence as authorization.
- Logging token values.
- Ignoring migration and device restore behavior.

# Debugging

- Check package platform setup and OS version support.
- Log exception type/category, not secret values.
- Test fresh install, app update, and device restore scenarios.
- Verify deletion on sign-out.
- Confirm the server rejects expired/revoked sessions.

# Interview Questions

1. What belongs in secure storage? Small secret material such as refresh tokens.
2. Is secure storage universally identical across platforms? No; native facilities and configuration vary.
3. Does a stored token prove the session is valid? No; validate or refresh it.
4. Can a secret embedded in a mobile binary be kept secret? Not reliably.

# Try It Yourself

1. Replace demo storage with a fake session repository.
2. Add an error UI when secure storage fails.
3. Test sign-out cleanup.
4. Write a threat model for the data stored by your app.

# Challenge Project

Implement secure session restoration around a mocked API. Store only a fake refresh token, validate it on startup, handle expiry and storage errors, and clear it on sign-out.

# Quick Revision Sheet

- Secure storage is for small secrets and uses platform facilities.
- Add package and platform setup.
- Never log or hard-code real credentials.
- Token presence is not token validity.
- Clear/revoke on sign-out and test migration.

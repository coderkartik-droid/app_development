# Introduction

`shared_preferences` is a Flutter plugin for storing small, simple key-value settings on a device. A key identifies a value, such as `darkMode`, and the value can be a `bool`, `int`, `double`, `String`, or `List<String>`.

It is intended for preferences and lightweight flags, not for a large database, private credentials, or complex records.

# Why this concept exists

Many useful app choices are small: theme, language, a dismissed tip, or a sorting option. A preference store offers a straightforward way to retain these values without creating a database schema.

# Real-world analogy

A preference store is a labeled drawer organizer. Each drawer has a unique label and holds one small item. It is convenient for a handful of settings, but not for a searchable archive.

# Internal Working

The plugin exposes asynchronous Dart methods that communicate with platform storage. The legacy `SharedPreferences.getInstance()` API returns a preferences object, and setter methods return a `Future<bool>`. Read the package documentation for platform behavior and newer APIs; do not assume every read is a fresh disk read in all API modes.

Preferences are not a transactional database and generally are not encrypted. Store only small, non-sensitive values. Await writes when correctness depends on completion, and keep defaults explicit.

# Syntax

```dart
final prefs = await SharedPreferences.getInstance();
await prefs.setString('language', 'en');
final language = prefs.getString('language') ?? 'en';
await prefs.remove('language');
```

The key is a string. Getters are synchronous after obtaining the preferences object, while obtaining and mutating the store use asynchronous operations.

# Complete Example 1

This runnable app saves a nickname and restores it when reopened. Add `shared_preferences` to the Flutter app's dependencies.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const PreferencesApp());

class PreferencesApp extends StatelessWidget {
  const PreferencesApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: NicknamePage());
}

class NicknamePage extends StatefulWidget {
  const NicknamePage({super.key});

  @override
  State<NicknamePage> createState() => _NicknamePageState();
}

class _NicknamePageState extends State<NicknamePage> {
  final _controller = TextEditingController();
  String _nickname = 'friend';
  bool _loading = true;
  String? _message;

  @override
  void initState() {
    super.initState();
    _load();
  }

  Future<void> _load() async {
    final prefs = await SharedPreferences.getInstance();
    final value = prefs.getString('nickname') ?? 'friend';
    if (!mounted) return;
    setState(() {
      _nickname = value;
      _loading = false;
    });
  }

  Future<void> _save() async {
    final value = _controller.text.trim();
    if (value.isEmpty) {
      setState(() => _message = 'Type a nickname first.');
      return;
    }
    final prefs = await SharedPreferences.getInstance();
    final saved = await prefs.setString('nickname', value);
    if (!mounted) return;
    setState(() {
      if (saved) {
        _nickname = value;
        _message = 'Saved.';
      } else {
        _message = 'The preference was not saved.';
      }
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Preferences')),
        body: _loading
            ? const Center(child: CircularProgressIndicator())
            : Padding(
                padding: const EdgeInsets.all(24),
                child: Column(
                  children: [
                    Text('Hello, $_nickname'),
                    TextField(
                      controller: _controller,
                      decoration: const InputDecoration(labelText: 'Nickname'),
                    ),
                    FilledButton(onPressed: _save, child: const Text('Save')),
                    if (_message != null) Text(_message!),
                  ],
                ),
              ),
      );
}
```

### Line-by-line explanation

- The imports provide Material widgets and the preferences plugin.
- `NicknamePage` owns the editable controller, displayed value, and loading/message state.
- `initState` starts a one-time read; `_load` reads the string or chooses the documented default.
- The `mounted` guard prevents an asynchronous result from updating a removed page.
- `_save` trims input, rejects empty text, obtains the store, and awaits `setString`.
- `setString` returns a boolean indicating whether the platform accepted the write; the UI displays the outcome.
- `dispose` releases the `TextEditingController`.
- The build method displays a loading indicator until the first read finishes, then shows the greeting and form.

### Widget tree

```text
MaterialApp → NicknamePage → Scaffold
                           └→ Column
                              ├→ greeting
                              ├→ TextField
                              ├→ Save button
                              └→ result message
```

### Execution flow

```text
Launch → obtain preferences → read nickname or default → render
Save tap → validate → await setString → show success/failure
```

### ASCII phone preview

```text
+---------------------------+
|        Preferences        |
+---------------------------+
| Hello, Sam                |
| Nickname                  |
| [ Sam                  ]  |
| [          Save         ] |
| Saved.                    |
+---------------------------+
```

### Example explanation

This is suitable for a small non-sensitive preference. The result message avoids claiming a successful save if the setter reports otherwise. If `_load` is removed, the stored nickname is never restored; if the await before updating state is removed, the screen may claim success too early.

**Common mistakes:** Do not store large JSON documents, passwords, or frequently changing collections here. Use a database or secure storage where appropriate.

**Professional use case:** Remembering a display option or language selection across launches.

# Complete Example 2

Use a preference key for a boolean onboarding flag:

```dart
final prefs = await SharedPreferences.getInstance();
await prefs.setBool('onboardingComplete', true);
final complete = prefs.getBool('onboardingComplete') ?? false;
```

The key uses a default of false so first launch shows onboarding. Treat this as a navigation preference, not as account authentication.

**Tree/flow:** startup service reads flag → route decision selects onboarding or home.

Removing the default leaves a nullable value that must be handled. Treating this flag as proof of a valid signed-in session is a security error.

# Complete Example 3

A numeric preference can remember a font scale:

```dart
await prefs.setDouble('fontScale', 1.0);
final scale = prefs.getDouble('fontScale') ?? 1.0;
```

Validate a loaded value against the app's supported range before applying it. Old or malformed data should not cause unreadable text or layout failure.

# Advanced Example

Wrap the plugin behind a typed settings repository. Centralize key names, defaults, validation, and migration from renamed keys. Keep storage details out of widgets, and test the repository with controlled preference instances rather than depending on device state.

# Under the Hood

The plugin serializes supported primitive values through platform-specific preference APIs. Its API is asynchronous for store access and mutation, while getters on the legacy object return cached typed values. This API does not offer relational queries, multi-record transactions, or encryption guarantees.

# Performance Notes

- Load settings once and expose them through app state.
- Avoid repeatedly acquiring and reading preferences in `build`.
- Keep values small.
- Await important writes, but do not block unrelated UI.
- Do not use preference writes as a high-frequency event log.

# Best Practices

- Use stable, namespaced keys such as `settings.themeMode`.
- Document defaults and validate loaded values.
- Isolate plugin code behind a repository.
- Handle failed writes explicitly.
- Use secure storage for secrets and a database for records.
- Review package migration guidance when adopting newer APIs.

# Common Mistakes

- Assuming preferences are encrypted or a database.
- Repeatedly reading from `build`.
- Storing a large list or serialized object blob.
- Ignoring nullable getters and defaults.
- Treating local flags as authentication proof.
- Ignoring setter results and errors.

# Debugging

- Confirm exact key spelling and expected value type.
- Check setter result and exceptions.
- Test a cold app restart.
- Clear app data only in a development/test environment when testing first-run behavior.
- Avoid logging sensitive preference values.

# Interview Questions

1. What is a key-value preference store best suited for? Small simple settings.
2. Which primitive types are commonly supported? Boolean, integer, double, string, and string list.
3. Is it suitable for passwords? No; use platform-backed secure storage.
4. Is it a relational database? No; it lacks relational queries and transactions.
5. Why centralize keys? To avoid typos, inconsistent defaults, and scattered storage policy.
6. Why test a cold restart? In-memory state can hide persistence failures.

# Try It Yourself

1. Store a preferred sort order and validate it when reading.
2. Add a repository method that returns a default theme value.
3. Show an error if a write reports failure.
4. Identify which settings should not be stored in preferences.

# Challenge Project

Create a typed settings repository for theme, language, and text scale. Validate every loaded value, expose settings to the UI, and add tests for defaults and invalid stored values.

# Quick Revision Sheet

- Preferences store small typed key-value settings.
- Add the package dependency before importing.
- Keys and defaults should be centralized.
- Reads/writes are tied to the local installation.
- Preferences are not encrypted, relational, or cross-device sync.
- Use a repository and test cold-start restoration.

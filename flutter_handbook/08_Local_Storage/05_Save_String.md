# Introduction

A string preference stores short text under a key. Common examples include a chosen language code, a last-selected tab name, or a display nickname.

# Why this concept exists

Small textual choices need to survive app restarts without requiring a full database. A string key-value pair is simple to write and read.

# Real-world analogy

It is a labeled sticky note in a drawer: quick to retrieve by its label, but not suitable for filing thousands of searchable documents.

# Internal Working

The app calls `setString(key, value)`. The plugin sends the value to platform storage. On a later read, `getString(key)` returns the stored string or `null` if the key is missing or has an incompatible type.

# Syntax

```dart
final prefs = await SharedPreferences.getInstance();
final ok = await prefs.setString('preferredLanguage', 'en');
final language = prefs.getString('preferredLanguage');
```

# Complete Example 1

This app saves and loads a short status message. Add `shared_preferences` to the project dependencies.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const StringPreferenceApp());

class StringPreferenceApp extends StatelessWidget {
  const StringPreferenceApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: StringPreferencePage());
}

class StringPreferencePage extends StatefulWidget {
  const StringPreferencePage({super.key});

  @override
  State<StringPreferencePage> createState() => _StringPreferencePageState();
}

class _StringPreferencePageState extends State<StringPreferencePage> {
  final _text = TextEditingController();
  String _saved = 'No saved message';
  bool _loading = true;

  @override
  void initState() {
    super.initState();
    _read();
  }

  Future<void> _read() async {
    final prefs = await SharedPreferences.getInstance();
    final value = prefs.getString('statusMessage');
    if (!mounted) return;
    setState(() {
      _saved = value ?? 'No saved message';
      _loading = false;
    });
  }

  Future<void> _save() async {
    final value = _text.text.trim();
    if (value.isEmpty) return;
    final prefs = await SharedPreferences.getInstance();
    final ok = await prefs.setString('statusMessage', value);
    if (!mounted) return;
    if (ok) setState(() => _saved = value);
  }

  @override
  void dispose() {
    _text.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Save a string')),
        body: Padding(
          padding: const EdgeInsets.all(24),
          child: _loading
              ? const Center(child: CircularProgressIndicator())
              : Column(
                  children: [
                    TextField(
                      controller: _text,
                      decoration: const InputDecoration(labelText: 'Message'),
                    ),
                    FilledButton(
                      onPressed: _save,
                      child: const Text('Save'),
                    ),
                    const SizedBox(height: 20),
                    Text(_saved),
                  ],
                ),
        ),
      );
}
```

### Line-by-line explanation

- Imports supply Flutter widgets and preferences.
- The state owns a text controller, displayed saved text, and loading flag.
- `_read` obtains the stored string and uses a user-friendly fallback if no key exists.
- `_save` trims input, skips empty strings, awaits persistence, and updates the screen only when the setter reports success.
- `dispose` releases the text controller.
- The UI presents a loading indicator before showing the input, save button, and stored value.

### Widget tree

```text
MaterialApp → Scaffold → Padding → Column
                                 ├→ TextField
                                 ├→ FilledButton
                                 └→ Text(saved value)
```

### Execution flow

```text
Start → readString → show stored text/default
Type → Save → trim → setString → update visible value
```

### ASCII phone preview

```text
+---------------------------+
|       Save a string       |
+---------------------------+
| Message                   |
| [ Remember milk        ]  |
| [          Save         ] |
| Remember milk             |
+---------------------------+
```

### Example explanation

`getString` is nullable, so the app explicitly handles a missing value. The input controller is disposed. Removing the setter means text is only in memory; removing the getter means a restart cannot restore it.

**Common mistakes:** Never store passwords as ordinary preference strings. Avoid using this API for large serialized content.

**Professional use case:** A chosen locale code or short display preference.

# Complete Example 2

Store a language code:

```dart
await prefs.setString('settings.locale', 'fr');
final code = prefs.getString('settings.locale') ?? 'en';
```

Validate the code against locales the app actually supports before creating a locale object.

# Complete Example 3

Store a selected category ID as a string:

```dart
await prefs.setString('lastCategoryId', category.id);
```

Persist identifiers rather than full mutable objects; load the current category record from the data repository when needed.

# Advanced Example

When renaming a key, support a controlled migration: read the new key first, otherwise read the old key, validate it, write the new key, and remove the old key only after a successful write.

# Under the Hood

The plugin bridges typed strings to platform preference storage. It does not index substrings or provide query capabilities. A missing key returns null, and a type mismatch should be treated as invalid persisted state rather than silently coerced.

# Performance Notes

- Keep text short.
- Do not repeatedly read preferences in widget build methods.
- Store large content in a file or database.
- Avoid rewriting an unchanged string on every keystroke unless a deliberate autosave policy exists.

# Best Practices

- Namespace keys.
- Trim and validate user input.
- Define default behavior for missing values.
- Await writes when subsequent UI depends on them.
- Keep sensitive text out of plain preferences.

# Common Mistakes

- Forgetting that reads can be null.
- Using inconsistent key spellings.
- Saving on every keystroke without debouncing.
- Storing secret tokens or passwords.
- Keeping full serialized models in a string key.

# Debugging

- Check key names and string type.
- Print non-sensitive operation outcomes in development.
- Restart the app to verify persistence.
- Check that the write finished before testing a read.
- Verify the value is not overwritten by another code path.

# Interview Questions

1. Why can `getString` return null? The key may not exist or the stored type may be wrong.
2. Why save an ID rather than a full object? IDs remain compact and let the repository load current data.
3. How should a key rename be handled? Migrate from the old key with validation and safe write ordering.
4. Where should passwords be stored? They generally should not be stored directly; use a secure authentication/token approach.

# Try It Yourself

1. Save a language code and validate supported locales.
2. Add a clear button that removes one key.
3. Add a maximum length to a nickname.
4. Test behavior when no value exists.

# Challenge Project

Build a “last opened project” setting that stores a project ID, validates it against current records, and falls back safely if the project has been deleted.

# Quick Revision Sheet

- `setString` writes; `getString` returns nullable text.
- Use small non-sensitive values.
- Validate and supply explicit defaults.
- Store IDs instead of large objects.
- Namespace keys and test cold restarts.

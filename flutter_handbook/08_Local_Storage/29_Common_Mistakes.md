# Introduction

Storage mistakes often remain invisible until an app is restarted, upgraded, used offline, or switched to another account. This chapter focuses on predictable failures and how to prevent them.

# Why this concept exists

Persisted data outlives the code version that created it. Incorrect assumptions about types, schema, security, or failures can cause data loss and privacy incidents.

# Real-world analogy

Changing a filing system without updating labels can make old records unreadable. Leaving a cabinet unlocked exposes information. Skipping a backup plan makes a small mistake permanent.

# Internal Working

The common failure categories are: wrong store for the data, incorrect lifecycle, schema incompatibility, unhandled asynchronous errors, concurrency races, and unclear ownership. Each category needs a specific test rather than a generic “works on my device” check.

# Syntax

A safe preference read names a fallback explicitly:

```dart
final value = prefs.getBool('settings.enabled') ?? false;
```

For databases, use bound parameters and versioned migrations.

# Complete Example 1

This app demonstrates safe handling of an optional saved integer. Add `shared_preferences` first.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const MistakesApp());

class MistakesApp extends StatelessWidget {
  const MistakesApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: SafeReadPage());
}

class SafeReadPage extends StatefulWidget {
  const SafeReadPage({super.key});

  @override
  State<SafeReadPage> createState() => _SafeReadPageState();
}

class _SafeReadPageState extends State<SafeReadPage> {
  int? _value;
  bool _loading = true;
  String? _error;

  @override
  void initState() {
    super.initState();
    _load();
  }

  Future<void> _load() async {
    try {
      final prefs = await SharedPreferences.getInstance();
      final stored = prefs.getInt('dailyGoal');
      if (!mounted) return;
      setState(() {
        _value = stored == null ? 5 : stored.clamp(1, 100).toInt();
        _loading = false;
      });
    } catch (error) {
      if (!mounted) return;
      setState(() {
        _loading = false;
        _error = 'Could not read the goal.';
      });
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Safe storage read')),
        body: Center(
          child: _loading
              ? const CircularProgressIndicator()
              : Text(_error ?? 'Daily goal: $_value'),
        ),
      );
}
```

### Line-by-line explanation

- The stored goal is nullable in storage, while the UI has explicit loading and error states.
- `_load` reads the value, uses a first-run default only when absent, and clamps stale values to the supported range.
- `try/catch` converts storage failure into a visible error rather than silently returning the default.
- `mounted` protects the asynchronous state update.
- The widget displays loading, error, or valid goal.

### Widget tree

```text
MaterialApp → SafeReadPage → Scaffold → Center
                                      ├→ loading
                                      └→ error or goal
```

### Execution flow

```text
Read → absent? default : validate stored value
     → error? show error
     → mounted? render result
```

### ASCII phone preview

```text
+---------------------------+
|    Safe storage read      |
+---------------------------+
| Daily goal: 12            |
+---------------------------+
```

### Example explanation

This example avoids conflating a missing key with a failed read and protects a widget from invalid old values. If the clamp is removed, a value no longer compatible with app constraints can create invalid UI. If the catch simply returns 5, storage failures become invisible.

**Common mistakes:** Defaults should represent genuine first-run policy, not disguise errors.

**Professional use case:** Load user settings safely across app versions.

# Complete Example 2

Unsafe SQL construction:

```dart
// Do not do this:
final sql = "SELECT * FROM users WHERE name = '$input'";
```

Use placeholders and bound parameters instead. The unsafe form lets input alter query syntax.

# Complete Example 3

A preference write should be awaited:

```dart
final saved = await prefs.setBool('tips.enabled', enabled);
if (!saved) {
  // Keep the previous UI state and report the failure.
}
```

Showing success before completion can mislead the user.

# Advanced Example

Classify failures as recoverable (retry), migration-required (upgrade data), corruption (restore/reset with consent), or security-sensitive (invalidate session). Do not use a broad catch to turn all failures into empty data.

# Under the Hood

Type mismatches and old serialized layouts are runtime compatibility issues. Async operations can complete after widget disposal or out of order. Database transactions help consistency but do not solve remote synchronization conflicts.

# Performance Notes

- Do not repeatedly initialize storage in widgets.
- Batch expensive database changes.
- Avoid oversized preference blobs.
- Add query indexes only for real query patterns.

# Best Practices

- Test restart, upgrade, offline, and account switching.
- Use unique key and field IDs.
- Await writes and surface failures.
- Parameterize SQL.
- Use the right store for sensitivity and structure.
- Keep a recovery/migration plan.

# Common Mistakes

- Assuming state in memory is persisted.
- Wrong preference type or key.
- Ignoring null and invalid old values.
- Creating controllers/DB connections in build.
- SQL injection through string concatenation.
- Storing tokens in preferences.
- Clearing all data to remove one key.

# Debugging

- Reproduce on a cold launch.
- Inspect key/type/schema and app version.
- Check async completion and widget lifecycle.
- Test every migration path.
- Review logs for accidental sensitive data.
- Use realistic data volume for performance bugs.

# Interview Questions

1. Why can old persisted data break a new app? The schema/meaning may have changed.
2. How do you prevent SQL injection? Parameterized queries.
3. Why should errors not default to empty data? It hides failure and may overwrite user state.
4. What is a read-modify-write race? Concurrent operations read the same old value and overwrite each other's updates.
5. Why test cold starts? In-memory state can mask persistence defects.

# Try It Yourself

1. Find a code sample that silently swallows an exception and redesign its state.
2. Add validation for a stale numeric preference.
3. Replace SQL interpolation with binding.
4. Write tests for missing, corrupt, and upgraded data.

# Challenge Project

Audit a local storage design for a hypothetical notes app. Identify at least five failure cases and propose tests and recovery UX for each.

# Quick Revision Sheet

- Missing, invalid, and failed are different states.
- Persisted schemas evolve.
- Await writes and do not fake success.
- Parameterize SQL.
- Use secure storage for secrets.
- Test cold start, upgrades, offline, and account switching.

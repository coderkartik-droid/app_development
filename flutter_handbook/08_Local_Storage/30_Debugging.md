# Introduction

Debugging storage means finding why a value is missing, stale, corrupt, slow, or incorrectly exposed. The most useful first step is identifying which store owns the value and tracing one operation from caller to storage and back.

# Why this concept exists

Storage defects may not appear during hot reload or a single app session. They often require a cold start, migration, permission failure, offline condition, or specific platform.

# Real-world analogy

When a package is missing, trace its route: who packed it, which shelf received it, and where it was expected. Repeating the same search without tracing the route rarely helps.

# Internal Working

A storage flow includes caller input, validation, serialization, I/O, persisted representation, read/decoding, state propagation, and UI. Debug each boundary with sanitized metadata. Never log tokens, passwords, or private records.

# Syntax

Useful development diagnostics:

```dart
debugPrint('Saving setting key=$key type=${value.runtimeType}');
```

Log identifiers and types only when they are non-sensitive. Avoid dumping full user content.

# Complete Example 1

This runnable preferences screen includes retryable load errors and clear states. Add `shared_preferences` to dependencies.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const StorageDebugApp());

class StorageDebugApp extends StatelessWidget {
  const StorageDebugApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: StorageDebugPage());
}

class StorageDebugPage extends StatefulWidget {
  const StorageDebugPage({super.key});

  @override
  State<StorageDebugPage> createState() => _StorageDebugPageState();
}

class _StorageDebugPageState extends State<StorageDebugPage> {
  String _status = 'Loading';
  String? _value;

  @override
  void initState() {
    super.initState();
    _load();
  }

  Future<void> _load() async {
    setState(() => _status = 'Loading');
    try {
      final prefs = await SharedPreferences.getInstance();
      final value = prefs.getString('debug.sample');
      if (!mounted) return;
      setState(() {
        _value = value;
        _status = value == null ? 'No value stored' : 'Value loaded';
      });
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Read failed; retry is available');
    }
  }

  Future<void> _write() async {
    try {
      final prefs = await SharedPreferences.getInstance();
      final ok = await prefs.setString('debug.sample', 'sample');
      if (!mounted) return;
      setState(() {
        _value = ok ? 'sample' : null;
        _status = ok ? 'Write completed' : 'Write did not complete';
      });
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Write failed');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Storage diagnostics')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_status),
              Text('Value: ${_value ?? '(none)'}'),
              FilledButton(onPressed: _load, child: const Text('Retry read')),
              OutlinedButton(onPressed: _write, child: const Text('Write test')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `_status` represents operation state separately from `_value`.
- `_load` makes the operation visible, reads a known test key, and distinguishes absent data from exceptions.
- `_write` awaits the setter and reports its result.
- Both async paths guard state updates with `mounted`.
- Retry and test-write buttons make behavior reproducible without restarting.

### Widget tree

```text
MaterialApp → StorageDebugPage → Scaffold → Center → Column
                                                   ├→ status
                                                   ├→ value
                                                   ├→ retry
                                                   └→ write test
```

### Execution flow

```text
Operation → success with value / missing key / exception
                       |
                       v
             explicit status shown to developer
```

### ASCII phone preview

```text
+---------------------------+
|   Storage diagnostics     |
+---------------------------+
| Value loaded              |
| Value: sample             |
| [      Retry read      ]  |
| [      Write test      ]  |
+---------------------------+
```

### Example explanation

The screen is a diagnostic tool, not a production data browser. It separates a missing preference from a thrown failure. If the catch is removed, users may see an unhandled exception; if the catch silently reports “no value,” the underlying fault is hidden.

**Common mistakes:** Do not include raw sensitive values in diagnostics or bug reports.

**Professional use case:** A development-only diagnostics panel for inspecting initialization and storage health.

# Complete Example 2

For database debugging, log query name, elapsed time, row count, and schema version:

```text
query=recent_tasks duration_ms=18 rows=20 schema=4
```

Keep bound user data out of logs.

# Complete Example 3

For migration debugging, test a copied database fixture from an earlier release and verify:

```text
old schema → migration → current version → expected records and indexes
```

Never run destructive reset commands against a user's production data as a debugging shortcut.

# Advanced Example

Build a reproducible matrix: fresh install, upgrade from each supported version, low storage, interrupted write, offline sync, sign-out/account switch, and app restart. Use test databases or fixtures and controlled preference instances.

# Under the Hood

Errors can originate in plugin initialization, native platform calls, filesystem access, database constraints, serialization adapters, or UI lifecycle. Trace the exception to its boundary; a storage library cannot explain an error swallowed by an unrelated catch block.

# Performance Notes

- Record operation durations in development.
- Profile database query plans and row counts.
- Avoid verbose logging in release builds.
- Use realistic data size for reproduction.

# Best Practices

- Log sanitized context and preserve stack traces in development.
- Keep diagnostics scoped to test/development builds.
- Reproduce on a cold start and target platform.
- Test migrations with real-format fixtures.
- Make retry safe and idempotent.

# Common Mistakes

- Clearing all app data as the first debugging step.
- Logging private values.
- Testing only hot reload.
- Hiding errors with broad catches.
- Assuming a successful UI update proves disk persistence.

# Debugging

- Confirm the exact key/table/box/path.
- Check type, schema, version, and store initialization.
- Distinguish missing from failed.
- Observe async ordering and mounted state.
- Test on each platform and after process restart.

# Interview Questions

1. How do you debug a missing preference? Trace write key/type/result, then cold-read the same key.
2. Why avoid logging values? They may contain sensitive user data.
3. What does a migration fixture test? Compatibility of old serialized data with the new schema.
4. Why is hot reload insufficient? It often preserves process memory and hides cold-start defects.

# Try It Yourself

1. Add operation duration logging without logging values.
2. Create a test for missing key and read exception.
3. Reproduce a migration from an older schema.
4. Check platform setup for a secure-storage plugin.

# Challenge Project

Create a storage diagnostics checklist for an app using preferences, database, files, and secure storage. Include safe logs, reproducible tests, and platform-specific checks.

# Quick Revision Sheet

- Trace caller → validation → store → decode → UI.
- Distinguish missing, invalid, and failed.
- Test cold starts and upgrades.
- Log metadata, never secrets or private content.
- Use fixtures and realistic data.
- Avoid destructive debugging shortcuts.

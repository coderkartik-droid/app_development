# Introduction

Reliable local storage is designed, not added as an afterthought. Good storage code has clear ownership, typed models, versioning, error handling, privacy boundaries, and tests.

# Why this concept exists

Persisted data survives code changes. A model that worked yesterday may be incompatible after an update, and a silent error can erase or misrepresent user work.

# Real-world analogy

A records office needs labels, access rules, retention schedules, and a way to handle old forms. A drawer full of unnamed papers is not a data strategy.

# Internal Working

Storage access belongs in a repository/service layer. The UI requests operations and consumes results; it should not scatter raw keys and SQL across widgets. Schema migrations transform existing data, while error states represent unavailable or invalid storage.

# Syntax

```dart
abstract interface class SettingsRepository {
  Future<bool> readDarkMode();
  Future<void> saveDarkMode(bool enabled);
}
```

# Complete Example 1

This runnable Flutter sample demonstrates a typed repository boundary with a simple in-memory implementation. A real app can replace it with preferences without changing the UI contract.

```dart
import 'package:flutter/material.dart';

abstract interface class SettingsRepository {
  Future<bool> readDarkMode();
  Future<void> saveDarkMode(bool enabled);
}

class MemorySettingsRepository implements SettingsRepository {
  bool _dark = false;

  @override
  Future<bool> readDarkMode() async => _dark;

  @override
  Future<void> saveDarkMode(bool enabled) async {
    _dark = enabled;
  }
}

void main() => runApp(const BestPracticesApp());

class BestPracticesApp extends StatefulWidget {
  const BestPracticesApp({super.key});

  @override
  State<BestPracticesApp> createState() => _BestPracticesAppState();
}

class _BestPracticesAppState extends State<BestPracticesApp> {
  final SettingsRepository _repository = MemorySettingsRepository();
  bool _dark = false;
  bool _ready = false;

  @override
  void initState() {
    super.initState();
    _load();
  }

  Future<void> _load() async {
    final value = await _repository.readDarkMode();
    if (!mounted) return;
    setState(() {
      _dark = value;
      _ready = true;
    });
  }

  Future<void> _save(bool value) async {
    await _repository.saveDarkMode(value);
    if (!mounted) return;
    setState(() => _dark = value);
  }

  @override
  Widget build(BuildContext context) => MaterialApp(
        theme: _dark ? ThemeData.dark() : ThemeData.light(),
        home: Scaffold(
          appBar: AppBar(title: const Text('Repository pattern')),
          body: Center(
            child: !_ready
                ? const CircularProgressIndicator()
                : SwitchListTile(
                    title: const Text('Dark mode'),
                    value: _dark,
                    onChanged: _save,
                  ),
          ),
        ),
      );
}
```

### Line-by-line explanation

- `SettingsRepository` defines the storage behavior the UI needs.
- `MemorySettingsRepository` is a test/demo fake; it does not persist after process restart.
- The app depends on the interface rather than storage package calls.
- `_load` restores the value from the repository and guards disposed state.
- `_save` awaits the repository operation before updating the root value.
- `MaterialApp` uses the value as the app theme.
- `_ready` separates loading from the default value.

### Widget tree

```text
BestPracticesApp → MaterialApp → Scaffold → SwitchListTile
                         |
                 SettingsRepository
```

### Execution flow

```text
UI → repository interface → storage implementation
Read result → app state → widgets
Toggle → repository write → app state changes
```

### ASCII phone preview

```text
+---------------------------+
|    Repository pattern     |
+---------------------------+
| Dark mode            [on] |
+---------------------------+
```

### Example explanation

The interface enables a production preferences implementation or a deterministic test fake. The sample repository is intentionally memory-only. If the UI calls a concrete plugin directly everywhere, changing stores and testing failures become harder.

If the await is removed, UI may claim the setting changed before persistence finishes. If errors are silently swallowed by a production repository, callers cannot communicate failure.

**Common mistakes:** Do not call this repository production-persistent until its implementation actually writes to disk.

**Professional use case:** A repository boundary isolates UI from storage library details and supports testing.

# Complete Example 2

Use a migration version:

```dart
@override
int get schemaVersion => 3;
```

This is one part of a migration strategy. The database must also define transformations from supported old versions.

# Complete Example 3

Represent an operation failure explicitly:

```dart
try {
  await repository.save(item);
} on StorageException catch (error) {
  // Show recoverable UI and report sanitized diagnostics.
}
```

Use the storage package's actual exception types. Do not catch every error and then pretend saving succeeded.

# Advanced Example

Define an inventory: data owner, sensitivity, store, schema version, retention, backup, sync, deletion, and recovery behavior. Use migrations and repository tests. Keep secrets out of logs, diagnostics, and analytics.

# Under the Hood

Repositories abstract I/O, state holders propagate results, and database engines or platform services provide persistence. Migrations transform previously serialized state so new code can safely read it.

# Performance Notes

- Open databases once in a managed lifecycle.
- Use indexes and pagination.
- Keep serialization bounded.
- Avoid redundant reads and writes.
- Profile cold startup and large queries.

# Best Practices

- Use typed repositories.
- Validate input and stored data.
- Await writes and surface failures.
- Version schemas and test migrations.
- Define backup, encryption, retention, and sync.
- Keep storage operations out of `build`.

# Common Mistakes

- Scattering raw keys/SQL through widgets.
- No migration plan.
- Swallowing errors.
- Storing all data in one untyped blob.
- Assuming local files are always safe and permanent.

# Debugging

- Add sanitized operation context and durations.
- Test empty, corrupt, old-version, and full-storage cases.
- Verify real cold start.
- Compare UI state with repository data.
- Inspect query plans and migration logs.

# Interview Questions

1. Why a repository? It separates UI/business rules from storage implementation.
2. What should a migration test cover? Old schema/data transformed to current representation.
3. Why validate data after reading? Stored state can be old, corrupted, or invalid under new constraints.
4. Why avoid broad catches? They hide failures and create false success.

# Try It Yourself

1. Replace the memory fake with shared preferences.
2. Add a failing repository fake and test UI error handling.
3. Write a storage inventory for an app.
4. Add a schema migration test plan.

# Challenge Project

Refactor a sample screen so all storage operations pass through a typed repository. Add fake-backed tests for success, missing values, failure, and schema-version assumptions.

# Quick Revision Sheet

- Centralize storage behind repositories.
- Validate both inputs and persisted values.
- Surface errors honestly.
- Migrate and test old data.
- Plan security, backup, retention, and deletion.
- Keep storage I/O out of widget build methods.

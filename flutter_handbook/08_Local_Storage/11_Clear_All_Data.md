# Introduction

Clearing all data is a broad operation that removes multiple stored values. It may mean clearing one feature's cache, resetting all preferences, removing an account's local database, or erasing all app data.

# Why this concept exists

Users may need to reset settings, remove cached content, sign out, or erase personal data. Broad clearing can also recover from corrupted local state, but careless use can destroy unrelated work.

# Real-world analogy

Clearing one drawer is different from emptying an entire home. Name the scope precisely and confirm before an irreversible action.

# Internal Working

`SharedPreferences.clear()` removes keys known to that preferences store. It is not a universal command that erases databases, files, secure storage, or remote data. A repository should own a scoped reset operation and enumerate resources intentionally.

# Syntax

```dart
final prefs = await SharedPreferences.getInstance();
final cleared = await prefs.clear();
```

# Complete Example 1

This app clears its preferences only after confirmation. Add `shared_preferences` as a dependency. Use this only when the app truly intends to reset all preference keys owned by this store.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const ClearDataApp());

class ClearDataApp extends StatelessWidget {
  const ClearDataApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: ClearDataPage());
}

class ClearDataPage extends StatefulWidget {
  const ClearDataPage({super.key});

  @override
  State<ClearDataPage> createState() => _ClearDataPageState();
}

class _ClearDataPageState extends State<ClearDataPage> {
  bool _darkMode = false;
  String _status = 'Preferences loaded';

  Future<void> _clear() async {
    final confirmed = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Reset preferences?'),
        content: const Text('All preferences in this store will be removed.'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Cancel'),
          ),
          FilledButton(
            onPressed: () => Navigator.pop(context, true),
            child: const Text('Reset'),
          ),
        ],
      ),
    );
    if (confirmed != true) return;
    final prefs = await SharedPreferences.getInstance();
    final cleared = await prefs.clear();
    if (!mounted) return;
    setState(() {
      _darkMode = false;
      _status = cleared ? 'Preferences reset' : 'Reset was not confirmed';
    });
  }

  @override
  Widget build(BuildContext context) => MaterialApp(
        theme: _darkMode ? ThemeData.dark() : ThemeData.light(),
        home: Scaffold(
          appBar: AppBar(title: const Text('Clear preferences')),
          body: Center(
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                Text(_status),
                FilledButton(
                  onPressed: _clear,
                  child: const Text('Reset preferences'),
                ),
              ],
            ),
          ),
        ),
      );
}
```

### Line-by-line explanation

- The root page owns the displayed setting and status.
- `_clear` presents an explicit confirmation dialog describing broad scope.
- Only a true confirmation continues.
- `prefs.clear()` removes preference keys in this store and returns a result.
- `mounted` guards the asynchronous completion.
- The UI resets its in-memory state only after the operation result is known.
- The nested `MaterialApp` demonstrates theme state; in a production app, keep one root `MaterialApp` and update its theme through the proper app-level state owner.

### Widget tree

```text
MaterialApp (theme from state)
└── Scaffold
    └── Center
        └── Column
            ├── status
            └── reset button
```

### Execution flow

```text
Reset → confirm → clear preferences → update in-memory settings/status
           └→ cancel → leave all data unchanged
```

### ASCII phone preview

```text
+---------------------------+
|   Clear preferences       |
+---------------------------+
| Preferences loaded        |
| [ Reset preferences ]     |
+---------------------------+
```

### Example explanation

The example demonstrates confirmation and scope, but real production code should keep a single root `MaterialApp` so theme state is owned centrally. `clear()` affects preferences only; it does not delete database files, secure values, or cloud data.

If confirmation is removed, a broad reset can happen accidentally. If other repositories' state is not refreshed, the UI may continue showing values that were removed from storage.

**Common mistakes:** Never call `clear()` as a generic cleanup unless all keys in that store are truly owned by the feature. Prefer namespaced keys and targeted removals.

**Professional use case:** A user-initiated “reset app preferences” action with a clear confirmation and documented scope.

# Complete Example 2

Clear only a feature's known keys:

```dart
await prefs.remove('reader.fontScale');
await prefs.remove('reader.lastBookId');
```

Targeted removal preserves unrelated app settings. Centralize these keys in a feature repository.

# Complete Example 3

A repository-level reset can coordinate different stores:

```dart
Future<void> resetLocalWorkspace() async {
  await taskDatabase.clearWorkspace();
  await fileCache.deleteWorkspaceFiles();
  await settingsRepository.resetWorkspaceSettings();
}
```

This is illustrative: methods and transaction boundaries depend on the app. Define partial-failure recovery because separate stores generally cannot participate in one atomic transaction.

# Advanced Example

For a privacy erase request, define the exact data inventory: preferences, secure tokens, database rows, attachment files, caches, analytics identifiers, and remote account data. Make operations idempotent so retries can safely continue after partial failure. Report completion only after required resources are addressed.

# Under the Hood

Preference `clear` removes key-value entries through the plugin's platform implementation. Databases and files use separate APIs and durability semantics. Clearing storage does not automatically clear OS-managed backups or server-side copies.

# Performance Notes

- Clear large caches in batches if deletion blocks visible work.
- Avoid repeatedly clearing/reinitializing stores on ordinary app startup.
- Keep reset paths out of hot UI frame callbacks.
- Track failures for multi-store reset operations.

# Best Practices

- Prefer a narrowly scoped reset.
- Confirm broad destructive actions.
- Document affected stores and whether remote data is included.
- Make reset operations idempotent and retryable.
- Refresh app state after clearing.
- Never use app-wide clear to delete one setting.

# Common Mistakes

- Assuming preferences `clear` erases every local store.
- Accidentally removing keys owned by another feature.
- Leaving in-memory state stale after disk deletion.
- Reporting success after partial failure.
- Confusing sign-out, reset, and account deletion.

# Debugging

- Enumerate keys/resources before and after a reset in a test environment.
- Verify unrelated settings remain after scoped reset.
- Test partial failure and retry.
- Restart the app to confirm defaults are restored.
- Check account data and remote data separately.

# Interview Questions

1. What does preferences `clear` remove? Preference keys in that store.
2. Does it clear SQLite and secure storage? No.
3. Why prefer targeted removal? It avoids deleting unrelated feature data.
4. How should multi-store erasure handle partial failure? Make it observable, idempotent, and retryable.

# Try It Yourself

1. Replace a broad clear with feature-specific key removal.
2. Add a confirmation dialog that states what is affected.
3. Build a reset checklist for a notes app.
4. Simulate failure in the second step of a multi-store reset.

# Challenge Project

Implement “reset reader settings” without deleting saved books or account tokens. Add tests proving that only reader-owned keys are removed.

# Quick Revision Sheet

- `clear()` is broad within the preference store, not universal.
- Prefer targeted, namespaced deletion.
- Separate reset, sign-out, cache clear, and account deletion.
- Refresh memory state after clearing.
- Plan for partial failure across different storage systems.

# Introduction

Deleting data removes one preference key, database record, file, or account-owned dataset. Deletion should be deliberate because lost user work may not be recoverable.

# Why this concept exists

Users need control over saved information, and apps need to remove obsolete cache entries and account data. Deletion also supports privacy and data-retention policies.

# Real-world analogy

Removing one labeled card is different from emptying an entire filing cabinet. Always identify the exact item and confirm broad deletion before executing it.

# Internal Working

Preferences can remove one key. Databases delete rows selected by a predicate. Filesystem deletion removes a specific path. Each operation should check its result and handle failure. Sensitive-data deletion may also require removing backups or remote copies according to policy.

# Syntax

```dart
final removed = await prefs.remove('obsoleteHint');
```

# Complete Example 1

This app removes one saved nickname preference. It asks for confirmation before deleting and updates UI only after the call reports success.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const DeletePreferenceApp());

class DeletePreferenceApp extends StatelessWidget {
  const DeletePreferenceApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: DeletePreferencePage());
}

class DeletePreferencePage extends StatefulWidget {
  const DeletePreferencePage({super.key});

  @override
  State<DeletePreferencePage> createState() => _DeletePreferencePageState();
}

class _DeletePreferencePageState extends State<DeletePreferencePage> {
  String? _nickname = 'Taylor';

  Future<void> _delete() async {
    final confirmed = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Remove nickname?'),
        content: const Text('This removes the saved nickname from this device.'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Cancel'),
          ),
          FilledButton(
            onPressed: () => Navigator.pop(context, true),
            child: const Text('Remove'),
          ),
        ],
      ),
    );
    if (confirmed != true) return;
    final prefs = await SharedPreferences.getInstance();
    final removed = await prefs.remove('nickname');
    if (!mounted) return;
    if (removed) setState(() => _nickname = null);
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Delete one value')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_nickname ?? 'No nickname saved'),
              FilledButton(
                onPressed: _nickname == null ? null : _delete,
                child: const Text('Delete nickname'),
              ),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `_nickname` represents the UI's current view of the stored value.
- `_delete` opens a confirmation dialog and receives `true`, `false`, or `null` if dismissed.
- The operation stops unless the user explicitly confirms.
- `SharedPreferences.remove` targets exactly one key.
- The result and mounted state are checked before updating the screen.
- The delete button is disabled when no nickname is displayed.
- `AlertDialog` explains the scope of the action and offers cancel/remove choices.

### Widget tree

```text
Scaffold → Center → Column
                   ├→ saved value
                   └→ delete button
Dialog (only while confirming) → Cancel / Remove
```

### Execution flow

```text
Delete tap → confirmation
                 ├→ cancel/dismiss → no mutation
                 └→ confirm → remove exact key → update UI on success
```

### ASCII phone preview

```text
+---------------------------+
|     Delete one value      |
+---------------------------+
| Taylor                    |
| [ Delete nickname ]       |
+---------------------------+
```

### Example explanation

The dialog limits accidental data loss, and the key is explicit. If confirmation is removed, a single mis-tap can erase the value. If the UI is cleared before the delete finishes, it may falsely indicate that persistent data was removed.

**Common mistakes:** Avoid broad clear operations when only one value should be removed. For important records, consider undo or a trash/archive flow.

**Professional use case:** Remove one locally remembered filter, preference, or draft entry.

# Complete Example 2

Delete a database record by ID:

```dart
await (database.delete(database.tasks)
      ..where((task) => task.id.equals(taskId)))
    .go();
```

Always include a predicate unless intentionally deleting all rows. Check the affected row count where the calling flow needs to distinguish “deleted” from “not found.”

# Complete Example 3

Delete a specific file after validating that it is an app-owned path:

```dart
final file = File(path);
if (await file.exists()) {
  await file.delete();
}
```

Do not accept arbitrary user-controlled paths without validation. A file may be shared, in use, or outside the app's managed directory.

# Advanced Example

For account deletion, local removal is only one part. Define remote account deletion, token revocation, local database cleanup, cached files, backups, and deletion confirmation. Make failures visible and resumable rather than claiming completion after only one layer succeeds.

# Under the Hood

Preference removal deletes a key; database deletion executes a predicate against rows; file deletion removes a filesystem entry. These are separate resources and are not automatically coordinated in one transaction.

# Performance Notes

- Database deletes should target indexed keys where possible.
- Clean up large file caches in bounded batches.
- Avoid blocking the UI while deleting many files.
- Do not log deleted sensitive content.

# Best Practices

- Make deletion scope clear.
- Confirm irreversible user-data removal.
- Prefer archive/undo for recoverable user content.
- Check operation results and surface failures.
- Remove associated cached and account-scoped data.
- Use precise predicates and app-owned paths.

# Common Mistakes

- Deleting all rows due to a missing predicate.
- Clearing UI before persistent deletion succeeds.
- Assuming local deletion removes cloud copies.
- Accepting arbitrary file paths.
- Failing to clear associated tokens or cached content.

# Debugging

- Verify the exact key, row ID, or path.
- Inspect affected-row result.
- Check permissions and file existence.
- Restart the app to ensure the data is really gone.
- Test cancel/dismiss behavior in confirmation dialogs.

# Interview Questions

1. How do you remove one preference? Call `remove` with the exact key.
2. Why include a database predicate? To limit deletion to intended records.
3. Is local deletion equivalent to account deletion? No; remote and related data may remain.
4. When is confirmation appropriate? For broad or irreversible user-data deletion.

# Try It Yourself

1. Add an undo action after removing a non-sensitive setting.
2. Delete one task by ID and display whether it existed.
3. Remove an associated attachment when deleting a note.
4. Write a deletion checklist for sign-out versus account removal.

# Challenge Project

Implement a “delete note and attachment” operation with a clear confirmation, safe file-path ownership, database consistency, and a user-visible error if either resource cannot be removed.

# Quick Revision Sheet

- `remove(key)` deletes one preference.
- Database deletes need precise predicates.
- File paths must be app-owned and validated.
- Local, remote, and backup deletion differ.
- Confirm irreversible actions and report partial failures.

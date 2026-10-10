# Introduction

Updating stored data means replacing an existing value or changing one field of a stored record. Preferences replace the value at a key; databases can update selected rows and columns.

# Why this concept exists

User settings and records change over time. A reliable update keeps the visible state and persisted state consistent, even when validation or storage fails.

# Real-world analogy

Updating a preference is like replacing the card in a labeled drawer. Updating a database row is like revising one field in a structured form while preserving the other fields.

# Internal Working

For a preference, `setX` overwrites the value at a key. For a database, an `UPDATE` statement identifies records with a predicate. A missing or overly broad predicate can produce incorrect results; use IDs and transactions for related changes.

# Syntax

```dart
await prefs.setString('displayName', newName);
```

# Complete Example 1

This app updates a saved city name and updates the visible label only if the setter reports success. Add the preferences package.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const UpdatePreferenceApp());

class UpdatePreferenceApp extends StatelessWidget {
  const UpdatePreferenceApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: CityPage());
}

class CityPage extends StatefulWidget {
  const CityPage({super.key});

  @override
  State<CityPage> createState() => _CityPageState();
}

class _CityPageState extends State<CityPage> {
  final _input = TextEditingController();
  String _city = 'Not set';

  Future<void> _updateCity() async {
    final nextCity = _input.text.trim();
    if (nextCity.isEmpty) return;
    final prefs = await SharedPreferences.getInstance();
    final saved = await prefs.setString('profile.city', nextCity);
    if (!mounted) return;
    if (saved) setState(() => _city = nextCity);
  }

  @override
  void dispose() {
    _input.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Update a preference')),
        body: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            children: [
              Text('City: $_city'),
              TextField(
                controller: _input,
                decoration: const InputDecoration(labelText: 'New city'),
              ),
              FilledButton(
                onPressed: _updateCity,
                child: const Text('Update'),
              ),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `_city` is the displayed value; `_input` captures the next one.
- `_updateCity` trims and rejects blank input, obtains preferences, and overwrites the namespaced key.
- It awaits the setter result and updates UI only on success.
- `mounted` prevents a late completion from updating a removed page.
- `dispose` releases the text controller.
- The UI displays the current value, input field, and update action.

### Widget tree

```text
MaterialApp → CityPage → Scaffold → Padding → Column
                                              ├→ Text
                                              ├→ TextField
                                              └→ Update button
```

### Execution flow

```text
Type new city → validate → setString(key, value)
                                |
                       success? ── yes → update UI
```

### ASCII phone preview

```text
+---------------------------+
| Update a preference       |
+---------------------------+
| City: Wellington          |
| New city                  |
| [ Wellington           ]  |
| [        Update        ]  |
+---------------------------+
```

### Example explanation

The setter replaces the old value for that key. If no value existed, it creates one. If the value is sensitive or part of a multi-field profile, a more appropriate storage layer may be required.

If the write is not awaited, UI can get ahead of disk. If the key is misspelled, a second preference is created instead of updating the original.

**Common mistakes:** Do not perform read-modify-write on preferences for concurrent multi-field state; use a transactional store where needed.

**Professional use case:** Update small independent settings such as display locale or selected sort mode.

# Complete Example 2

Use a database update with a stable primary key. With Drift, a generated table API can express an update:

```dart
await (database.update(database.tasks)
      ..where((task) => task.id.equals(taskId)))
    .write(TasksCompanion(title: Value(newTitle)));
```

The predicate selects the row, and a companion marks the field being changed. Verify the affected row count if the caller needs to know whether the record existed.

# Complete Example 3

When updating multiple related rows, use a database transaction:

```dart
await database.transaction(() async {
  await repository.updateAccount(account);
  await repository.updatePreferences(account.id, preferences);
});
```

The transaction ensures the group commits or rolls back together according to the database engine and API.

# Advanced Example

For optimistic updates, update local state immediately, persist the change, then reconcile with the remote result. On failure, either roll back or mark the change pending for retry. Use version fields or conflict policies when multiple writers can update the same record.

# Under the Hood

Preference setters replace one key's value. SQL database updates evaluate a `WHERE` predicate and modify matching rows, often inside implicit or explicit transactions. ORM wrappers generate parameterized queries; never interpolate untrusted input into raw SQL.

# Performance Notes

- Update only changed columns.
- Batch related writes where safe.
- Index lookup predicates for large tables.
- Avoid rewriting full serialized objects for one field.
- Debounce rapid settings changes.

# Best Practices

- Validate before writing.
- Use stable keys/IDs.
- Await storage completion before claiming success.
- Use transactions for related data.
- Define optimistic rollback and conflict behavior.
- Use parameterized queries.

# Common Mistakes

- Updating the wrong key or row.
- Omitting a `WHERE` predicate.
- Replacing a whole record and losing untouched fields.
- Assuming preference updates are transactional across keys.
- Updating UI without considering write failure.

# Debugging

- Log key/record ID and affected row count, never secret values.
- Check setter result or database update count.
- Verify predicates and schema field mapping.
- Reproduce failure during a multi-row operation.
- Confirm UI refreshes from the authoritative repository.

# Interview Questions

1. How does preference update work? Set the same key to replace its value.
2. Why use a stable primary key? To target one record unambiguously.
3. Why is `WHERE` important? Without it, an update may affect every row.
4. What does a transaction provide? All-or-nothing grouping for related database operations.
5. What is optimistic update? Updating local UI/data before remote confirmation, with reconciliation.

# Try It Yourself

1. Add input validation to a preference update.
2. Update one database column while preserving the others.
3. Add an affected-row check.
4. Simulate a failed optimistic write and roll back.

# Challenge Project

Implement editing a task title and completion flag. Use a transaction if both fields and an audit record must change together; handle missing IDs and failed updates explicitly.

# Quick Revision Sheet

- Preference setter replaces one key's value.
- Database updates should target stable IDs with predicates.
- Use transactions for related writes.
- Await writes and report failure honestly.
- Avoid whole-record replacement when only one field changes.

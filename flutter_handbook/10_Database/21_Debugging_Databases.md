# Introduction

Database debugging is the process of tracing a problem from the app's requested operation to the actual file, schema, query, and returned data. A blank screen may be a UI-state issue, a failed migration, or simply a query that matched no rows.

# Why this concept exists

Persistence bugs can survive hot reload and appear device-specific. Inspecting the database directly and separating engine errors from UI rendering makes diagnosis faster.

# Real-world analogy

When a package is missing, check the address label, dispatch record, and delivery receipt in order instead of blaming the warehouse immediately.

# Internal Working

A database request passes through the repository and package to SQLite. Errors complete the awaited future; if ignored, the UI may remain stale. Debug by verifying file path, schema, operation, result, and state update in sequence.

```text
correct file? -> expected schema? -> SQL/args? -> rows/count? -> state? -> widget?
```

# Syntax

```dart
try {
  final rows = await db.query('tasks');
  debugPrint('Loaded ${rows.length} task rows');
} on DatabaseException catch (error, stackTrace) {
  debugPrint('Task query failed: $error\n$stackTrace');
  rethrow;
}
```

# Complete Example 1

This Flutter app reports database initialization and query failures instead of disguising them as an empty list. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openDebugDb() async {
  final file = p.join(await getDatabasesPath(), 'debugging_demo.db');
  debugPrint('Opening database: $file');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE tasks (id INTEGER PRIMARY KEY, title TEXT NOT NULL)',
    );
  });
}

void main() => runApp(const MaterialApp(home: DebugPage()));

class DebugPage extends StatefulWidget {
  const DebugPage({super.key});
  @override
  State<DebugPage> createState() => _DebugPageState();
}

class _DebugPageState extends State<DebugPage> {
  late final Future<Database> _db = openDebugDb();
  String _status = 'Loading';

  Future<void> _load() async {
    try {
      final db = await _db;
      final rows = await db.query('tasks');
      debugPrint('Task query returned ${rows.length} rows');
      if (mounted) setState(() => _status = 'Loaded ${rows.length} tasks');
    } on DatabaseException catch (error, stackTrace) {
      debugPrint('SQLite failure: $error\n$stackTrace');
      if (mounted) setState(() => _status = 'Database error');
    } catch (error, stackTrace) {
      debugPrint('Unexpected failure: $error\n$stackTrace');
      if (mounted) setState(() => _status = 'Unexpected error');
    }
  }

  @override
  void initState() {
    super.initState();
    _load();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Database diagnostics')),
        body: Center(child: Text(_status)),
        floatingActionButton: FloatingActionButton(
          onPressed: _load,
          child: const Icon(Icons.refresh),
        ),
      );
}
```

**Line-by-line guide:** Imports provide UI, paths, and SQLite. `openDebugDb` prints the resolved path in development, opens the stable file, and creates the initial table. The state retains one database future and a status. `_load` awaits open and query, logs a row count, and updates UI. `DatabaseException` is handled separately from unexpected errors; both are visible and retain diagnostic stack traces in logs. `mounted` protects each asynchronous state update. `initState` loads; the refresh button retries. Widget tree: `MaterialApp -> DebugPage -> Scaffold -> AppBar + Center/Text + FAB`.

**Database flow:** `open path -> schema callback -> query -> log row count OR exception -> status -> UI`.

**Execution:** A successful query displays its count; a missing table or open failure displays an error rather than “zero tasks.” Removing the catch leaves a failed future unhandled; returning an empty list on error would incorrectly mask the cause.

```text
+----------------------+
| Database diagnostics |
|                      |
| Loaded 0 tasks       |
|                 [R]  |
+----------------------+
```

**Common mistakes:** Logging private row contents, treating every failure as no data, and debugging the wrong database file.

**Professional use case:** Development diagnostics and production-safe error reporting for repositories.

# Complete Example 2

Inspect columns:

```dart
final columns = await db.rawQuery('PRAGMA table_info(tasks)');
for (final column in columns) {
  debugPrint('Column: ${column['name']} type=${column['type']}');
}
```

This validates the schema actually opened, not the schema developers think exists. Avoid dumping user data into logs.

# Complete Example 3

Check foreign-key violations:

```dart
final violations = await db.rawQuery('PRAGMA foreign_key_check');
```

An empty result means no current violations were found. It does not prove every connection has enforcement enabled; inspect `PRAGMA foreign_keys` as well.

# Advanced Example

Use a copied test database to reproduce upgrade issues. Record its schema version, migration path, and safe fixture data. Do not delete a user's database as a debugging shortcut.

# Under the Hood

SQLite errors surface through package exceptions. A valid query can still return zero rows; that is distinct from an exception. The UI needs separate empty, loading, and failure states.

# Performance Notes

Development logging is useful but can leak data or slow large flows. Log row counts, operation names, durations, and non-sensitive IDs—not full tables.

# Best Practices

- Trace the exact path and schema version.
- Reproduce with a minimal query and known fixture.
- Distinguish empty results from errors.
- Keep sensitive payloads out of logs.

# Common Mistakes

- Swallowing exceptions in a broad catch.
- Deleting the live database before taking a safe backup.
- Assuming hot reload resets persistent data.
- Inspecting a different app/device database than the one under test.

# Debugging

Use `PRAGMA table_info`, `PRAGMA user_version`, and `PRAGMA foreign_key_check`. Confirm filters/bound args and mutation counts; then confirm application state received the result.

# Interview Questions

1. **How distinguish empty from failed query?** Empty is a successful zero-row result; failure throws/completes with an error.
2. **Why inspect the database path?** Multiple files can exist if path construction differs.
3. **What does `foreign_key_check` reveal?** Existing rows violating declared foreign keys.
4. **Why avoid logging rows?** They may contain sensitive user data.

# Try It Yourself

Intentionally query a missing table, capture the error, and show an error state distinct from an empty list.

# Challenge Project

Create a database diagnostics mode that reports schema version, table names, row counts, and migration errors without exposing record contents.

# Quick Revision Sheet

- Trace path -> schema -> query -> result/error -> state -> UI.
- Empty result is not an exception.
- Inspect schema and foreign keys with PRAGMA.
- Log metadata, not private records.

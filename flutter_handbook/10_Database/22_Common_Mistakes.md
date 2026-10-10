# Introduction

Database mistakes often come from treating persistent storage like a temporary Dart list. A database has file identity, schema, asynchronous operations, constraints, and lifecycle; getting those boundaries wrong can lose data or display stale information.

# Why this concept exists

Learning common failure patterns helps prevent accidental table-wide changes, data loss during upgrades, and performance problems that only appear on real devices with realistic data.

# Real-world analogy

A database is an archive, not a whiteboard. Changing the filing rules affects old folders too, and erasing the wrong drawer may be irreversible.

# Internal Working

Most failures occur at one of four boundaries: file opening, schema/versioning, query semantics, or UI state/lifecycle. Trace each database future to its result or exception and confirm which file was used.

```text
wrong path | wrong schema | wrong SQL/predicate | stale UI | unhandled error
```

# Syntax

```dart
await db.delete(
  'tasks',
  where: 'id = ?',
  whereArgs: [taskId],
);
```

The predicate is essential; without it, every row may be deleted.

# Complete Example 1

This runnable Flutter sample demonstrates a safe, scoped delete and refresh. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openSafeDb() async {
  final file = p.join(await getDatabasesPath(), 'mistakes_demo.db');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE tasks (id INTEGER PRIMARY KEY AUTOINCREMENT, title TEXT NOT NULL)',
    );
  });
}

void main() => runApp(const MaterialApp(home: SafeDeletePage()));

class SafeDeletePage extends StatefulWidget {
  const SafeDeletePage({super.key});
  @override
  State<SafeDeletePage> createState() => _SafeDeletePageState();
}

class _SafeDeletePageState extends State<SafeDeletePage> {
  late final Future<Database> _db = openSafeDb();
  List<Map<String, Object?>> _rows = [];

  Future<void> _load() async {
    final rows = await (await _db).query('tasks', orderBy: 'id');
    if (mounted) setState(() => _rows = rows);
  }

  Future<void> _add() async {
    await (await _db).insert('tasks', {'title': 'Keep this record'});
    await _load();
  }

  Future<void> _delete(int id) async {
    final count = await (await _db).delete(
      'tasks',
      where: 'id = ?',
      whereArgs: [id],
    );
    if (count != 1) {
      throw StateError('Expected one deleted task, got $count');
    }
    await _load();
  }

  @override
  void initState() {
    super.initState();
    _load();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Scoped database changes')),
        body: ListView(
          children: _rows
              .map((row) => ListTile(
                    title: Text(row['title']! as String),
                    trailing: IconButton(
                      icon: const Icon(Icons.delete),
                      onPressed: () => _delete(row['id']! as int),
                    ),
                  ))
              .toList(),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: _add,
          child: const Icon(Icons.add),
        ),
      );
}
```

**Line-by-line guide:** Imports supply the Flutter UI and database utilities. `openSafeDb` uses a stable path and creates a task table. The page caches one open future. `_load` reads rows and updates state if mounted. `_add` inserts then reloads. `_delete` binds a specific ID, checks the affected-row count, and reloads. The row map is converted to an integer ID for the button callback. `initState` starts the initial read. Widget tree: `MaterialApp -> SafeDeletePage -> Scaffold -> ListView/ListTile/IconButton + FAB`.

**Database flow:** `delete tap -> ID placeholder -> one-row DELETE -> affected count -> SELECT -> UI`.

**Execution:** The selected row is removed. Removing `where` clears all tasks; removing the count check can conceal an outdated or incorrect ID. Removing refresh leaves the deleted row visible until another read.

```text
+----------------------+
| Scoped changes       |
|----------------------|
| Keep this record [x] |
|                 [+]  |
+----------------------+
```

**Common mistakes:** This example highlights missing predicates, stale UI, and unchecked results. Never test destructive operations on valuable data.

**Professional use case:** Production delete operations should be narrowly scoped, confirmed when appropriate, and covered by repository tests.

# Complete Example 2

Do not interpolate values:

```dart
await db.query('tasks', where: 'title = ?', whereArgs: [userEnteredTitle]);
```

Bound arguments avoid SQL injection and quoting bugs. A placeholder cannot safely represent a table or column identifier; keep identifiers static or choose from a strict allowlist.

# Complete Example 3

Increment schema version when a released schema changes and migrate existing rows. Editing `onCreate` alone affects only new database files, not installations that already created the older schema.

# Advanced Example

Test negative cases: nonexistent IDs, duplicate keys, missing required fields, migration failures, large result sets, and a widget disposed while a query is pending. Verify that errors remain visible and state is not updated after disposal.

# Under the Hood

SQLite persists independently of Flutter widget state. Hot reload usually preserves the database file, so code changes do not reset stored rows. Opening another path can make data seem lost even though the original file remains.

# Performance Notes

Avoid unbounded reads, per-row queries, and indexes without measured benefit. Large synchronous mapping after a query can also delay rendering.

# Best Practices

- Bind values and scope every mutation.
- Version every schema evolution.
- Distinguish errors from empty results.
- Test with old database files and realistic data.
- Preserve user data and make destructive behavior intentional.

# Common Mistakes

- Reopening a different filename after refactoring.
- Expecting hot reload to clear persistent data.
- Deleting all rows by omitting a predicate.
- Swallowing exceptions or turning them into empty data.
- Relying only on UI validation instead of schema constraints.

# Debugging

Check the resolved path, schema version, query arguments, affected count, and final rows. Reproduce on a copy of the database before attempting repairs.

# Interview Questions

1. **Why does editing `onCreate` not update installed apps?** It is invoked only for a new database.
2. **What is the danger of a missing delete predicate?** It can remove every row from the table.
3. **Why do values use placeholders?** To bind data safely and correctly.
4. **Why might UI show old data after a successful write?** App state was not refreshed or reconciled.

# Try It Yourself

Review a CRUD sample and mark every operation that could affect more than the intended row.

# Challenge Project

Write tests that catch a missing update/delete predicate, verify migrations preserve records, and distinguish empty from error states.

# Quick Revision Sheet

- File identity matters; use stable paths.
- Database changes do not automatically update Flutter state.
- Mutations need predicates and affected-row checks.
- Existing files need migrations, not edited `onCreate` callbacks.

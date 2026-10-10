# Introduction

This cheat sheet is a compact reference for Flutter database work. Use it to recall common SQLite syntax, `sqflite` patterns, relationships, transactions, and migration rules.

# Why this concept exists

Database features cross schema, query, package, state, and lifecycle boundaries. A short checklist helps avoid omissions during implementation and review.

# Real-world analogy

It is a mechanic's quick card: the full manual teaches each subsystem, while the card helps verify the key steps before a drive.

# Internal Working

Flutter requests database work asynchronously through a package. SQLite executes SQL and returns rows or mutation results; app state then drives the UI.

```text
Widget -> Repository -> Database package -> SQLite file
       <- state       <- typed results    <- rows/count/errors
```

# Syntax

```sql
CREATE TABLE tasks (id INTEGER PRIMARY KEY, title TEXT NOT NULL);
CREATE INDEX idx_tasks_title ON tasks(title);
SELECT id, title FROM tasks WHERE id = ?;
```

# Complete Example 1

Minimal complete Flutter example: create/open a database, insert one record, and show a count. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openQuickDb() async {
  final file = p.join(await getDatabasesPath(), 'database_cheat_sheet.db');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE tasks (id INTEGER PRIMARY KEY AUTOINCREMENT, title TEXT NOT NULL)',
    );
  });
}

void main() => runApp(const MaterialApp(home: CheatSheetPage()));

class CheatSheetPage extends StatefulWidget {
  const CheatSheetPage({super.key});
  @override
  State<CheatSheetPage> createState() => _CheatSheetPageState();
}

class _CheatSheetPageState extends State<CheatSheetPage> {
  late final Future<Database> _db = openQuickDb();
  int _count = 0;

  Future<void> _refresh() async {
    final count = Sqflite.firstIntValue(
          await (await _db).rawQuery('SELECT COUNT(*) FROM tasks'),
        ) ??
        0;
    if (mounted) setState(() => _count = count);
  }

  Future<void> _add() async {
    await (await _db).insert('tasks', {'title': 'Quick reference'});
    await _refresh();
  }

  @override
  void initState() {
    super.initState();
    _refresh();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Database quick sheet')),
        body: Center(child: Text('Tasks: $_count')),
        floatingActionButton: FloatingActionButton(
          onPressed: _add,
          child: const Icon(Icons.add),
        ),
      );
}
```

**Line-by-line guide:** Imports provide Flutter UI, path joining, and `sqflite`. `openQuickDb` uses a stable file and creates a table on first opening. The page caches the database future. `_refresh` asks SQLite for a count and updates state only if mounted. `_add` inserts a row and refreshes. `initState` loads the existing count. `build` displays it with a button. Widget tree: `MaterialApp -> CheatSheetPage -> Scaffold -> AppBar + Center/Text + FAB`.

**Database flow:** `stable open -> schema -> insert -> COUNT query -> state -> display`.

**Execution:** The count increments after every tap and persists across relaunch. Removing table initialization makes the query fail; removing refresh leaves the count stale.

```text
+----------------------+
| Database quick sheet |
|                      |
| Tasks: 1             |
|                 [+]  |
+----------------------+
```

**Common mistakes:** Do not run this initialization in `build`, and do not omit a migration when the schema changes.

**Professional use case:** Keep this sequence as a checklist when reviewing a new database-backed feature.

# Complete Example 2

**CRUD quick map:** `insert(table, values)`, `query(table, where, whereArgs)`, `update(table, values, where, whereArgs)`, and `delete(table, where, whereArgs)`. Always scope update/delete and bind values. Query methods return rows; mutations return an inserted ID or affected-row count.

# Complete Example 3

**Relationship quick map:** one-to-many places the foreign key on the many-side table; many-to-many uses a junction table; one-to-one uses a unique foreign key or shared primary/foreign key. Enable SQLite foreign keys and test delete behavior.

# Advanced Example

**Migration quick map:** increment schema version, preserve a sequence of upgrades, update fresh-install schema, test every supported old version, and verify data. **Transaction quick map:** use `db.transaction((txn) async { ... })`, use `txn` inside, and let errors propagate for rollback.

# Under the Hood

SQLite is an embedded relational engine, not a network server. Its indexes and query planner affect performance; constraints and transactions protect integrity. Flutter's widget tree does not automatically observe raw `sqflite` writes.

# Performance Notes

Project needed columns, filter and aggregate in SQL, paginate large results, use indexes only for measured paths, and avoid N+1 reads.

# Best Practices

- Stable database path and centralized opener.
- Typed repository boundary and parameterized SQL.
- Constraints, foreign keys, and transactions.
- Versioned migrations and integration tests.
- Explicit empty/loading/error states and safe logging.

# Common Mistakes

- SQL interpolation or missing mutation predicates.
- Editing only `onCreate` after release.
- Treating UI state as database state.
- Assuming data is encrypted, backed up, or synchronized automatically.

# Debugging

Inspect resolved file, `PRAGMA user_version`, `PRAGMA table_info`, foreign-key status, query plan, bound arguments, and affected counts.

# Interview Questions

1. **SQLite vs `sqflite`?** SQLite is the engine; `sqflite` is a Flutter/Dart integration package.
2. **What is ACID?** Atomicity, consistency, isolation, durability.
3. **Why add an index?** To optimize a measured access path, with storage/write trade-offs.
4. **How preserve data across app versions?** Tested, versioned migrations.

# Try It Yourself

Implement a table with a unique field, a scoped update, a transaction, and one migration. Test empty and failure behavior.

# Challenge Project

Build a database-backed app using the section's patterns: schema, repository, relationships, queries, migrations, performance, and tests.

# Quick Revision Sheet

- CRUD: insert, query, update, delete.
- Use placeholders and predicates.
- Transactions protect related writes.
- Migrations evolve installed databases.
- Measure queries; test persistent data and upgrade paths.

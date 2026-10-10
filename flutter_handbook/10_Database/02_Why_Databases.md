# Introduction

An app needs a database when it must preserve structured records, query them efficiently, or maintain relationships between records. For example, an expense tracker must retain transactions and filter them by date, category, or amount.

# Why this concept exists

In-memory lists vanish when an app closes, while ad hoc files make searching, consistency, and concurrent edits hard. A database provides a tested engine for durable storage, constraints, indexes, and atomic changes.

# Real-world analogy

A database is like a filing office with labeled drawers, unique file numbers, and an index. You can locate one record or summarize a whole category without manually scanning every paper.

# Internal Working

The app sends database operations through a package to the engine. The engine checks constraints, reads or updates storage pages, and returns results. Dart awaits the result and updates the state that drives widgets.

```text
User intent -> repository -> database engine -> durable file
             <- query result <- rows / count / error
                         -> state -> Flutter UI
```

# Syntax

```dart
final changed = await db.update(
  'tasks',
  {'done': 1},
  where: 'id = ?',
  whereArgs: [taskId],
);
```

`changed` is the number of affected rows, useful for distinguishing a successful match from an outdated ID.

# Complete Example 1

This complete Flutter example stores simple tasks with `sqflite`. Add `sqflite` and `path` to `pubspec.yaml`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openTaskDb() async {
  final dbPath = p.join(await getDatabasesPath(), 'why_tasks.db');
  return openDatabase(dbPath, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE tasks (id INTEGER PRIMARY KEY AUTOINCREMENT, title TEXT NOT NULL, done INTEGER NOT NULL DEFAULT 0)',
    );
  });
}

void main() => runApp(const MaterialApp(home: TaskPage()));

class TaskPage extends StatefulWidget {
  const TaskPage({super.key});
  @override
  State<TaskPage> createState() => _TaskPageState();
}

class _TaskPageState extends State<TaskPage> {
  late final Future<Database> _db = openTaskDb();
  List<Map<String, Object?>> _tasks = [];

  Future<void> _refresh() async {
    final rows = await (await _db).query('tasks', orderBy: 'id DESC');
    if (mounted) setState(() => _tasks = rows);
  }

  Future<void> _addTask() async {
    await (await _db).insert('tasks', {'title': 'Learn database basics'});
    await _refresh();
  }

  @override
  void initState() {
    super.initState();
    _refresh();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Persistent tasks')),
        body: ListView(
          children: _tasks
              .map((row) => ListTile(title: Text(row['title']! as String)))
              .toList(),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: _addTask,
          child: const Icon(Icons.add),
        ),
      );
}
```

**Line-by-line guide:** Imports provide UI, path joining, and SQLite. `openTaskDb` creates a stable file path and opens schema version 1. `CREATE TABLE` gives each task a generated identity, requires a title, and defaults `done` to false as integer zero. `main` launches the page. The state retains the open future and query results. `_refresh` reads rows in descending ID order and updates state only while mounted. `_addTask` inserts a title, then refreshes. `initState` starts the initial read. The widget tree is `MaterialApp -> TaskPage -> Scaffold -> ListView/ListTile + FloatingActionButton`.

**Database flow:** `tap -> INSERT task -> persisted row -> SELECT tasks -> setState -> render list`.

**Execution:** Launch creates the table if needed; each plus press adds a durable row. If the table creation callback is removed, the first insert fails. If `_refresh` after insert is removed, the row is saved but not immediately visible.

```text
+----------------------+
| Persistent tasks     |
|----------------------|
| Learn database...    |
|                 [+]  |
+----------------------+
```

**Common mistakes:** Treating `done` as a Dart boolean in SQLite without converting it, reading all records for simple counts, and swallowing database errors.

**Professional use case:** A task list remains useful offline and restores its records on a later app launch.

# Complete Example 2

Use SQL to filter at the source:

```dart
final openTasks = await db.query(
  'tasks',
  where: 'done = ?',
  whereArgs: [0],
);
```

`where` defines the predicate; `whereArgs` binds its value. SQLite returns only unfinished records, reducing Dart-side work. Removing the predicate loads completed tasks too; interpolating values into the predicate is unsafe.

# Complete Example 3

Delete one task and inspect the affected count:

```dart
final deleted = await db.delete(
  'tasks',
  where: 'id = ?',
  whereArgs: [taskId],
);
```

The `id = ?` predicate limits the action to one row. Without `where`, the operation deletes every task. Refresh the UI after deletion.

# Advanced Example

Add a composite index when measured queries repeatedly filter by completion and order by ID:

```sql
CREATE INDEX idx_tasks_done_id ON tasks(done, id);
```

An index speeds matching reads at the cost of additional storage and slower writes. Confirm benefit with representative data and query plans.

# Under the Hood

The database engine maps logical rows to pages and maintains indexes and journaling for consistency. Constraints reject invalid writes before they become durable. Flutter's rendering pipeline remains separate from this work.

# Performance Notes

Indexes are not free. Avoid selecting unused columns, fetching unbounded histories, or performing repeated per-row queries when a join or batch is appropriate.

# Best Practices

- Pick a database based on query and relationship needs.
- Define expected offline behavior and data-retention policy.
- Keep schema and migrations under version control.
- Measure using realistic record counts.

# Common Mistakes

- Assuming a database is needed for every single value.
- Storing relational collections as one serialized string without a query need analysis.
- Deleting all rows accidentally by omitting `where`.
- Updating UI before a write has completed.

# Debugging

Inspect the file, schema, row counts, and query arguments. Reproduce with empty, typical, and large datasets; verify the row count returned by mutations.

# Interview Questions

1. **When is a database useful?** When structured data must persist and be queried or updated.
2. **What does a constraint do?** It rejects values that violate declared data rules.
3. **What is an index trade-off?** Faster reads for matching access paths, with extra storage and write cost.
4. **Why check affected rows?** It confirms the predicate matched records.

# Try It Yourself

Add a completed filter and a task-count query. Compare reading all rows and counting in SQL.

# Challenge Project

Create an offline task manager with status filters, due dates, and a search field. Ensure each action persists across restart.

# Quick Revision Sheet

- Databases solve durability, structure, querying, and consistency.
- SQL can filter and aggregate before results reach Dart.
- Constraints protect data invariants.
- Indexes improve selected access paths, not every operation.
- UI refresh must follow database operations.

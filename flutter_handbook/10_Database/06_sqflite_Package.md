# Introduction

`sqflite` is a Flutter package that exposes SQLite database operations through Dart `Future`s. It offers helpers such as `query`, `insert`, `update`, and `delete`, plus raw SQL execution and transactions.

# Why this concept exists

Calling SQLite's native APIs directly from every screen would spread platform details and SQL throughout the app. `sqflite` provides a consistent Dart-facing interface for mobile database work.

# Real-world analogy

`sqflite` is a bilingual clerk between Flutter and SQLite: Flutter describes a query in Dart terms, and the clerk relays it to the database engine and translates the result back.

# Internal Working

The app awaits a package method. The package sends work to the platform database implementation and returns rows or a mutation count. These operations are asynchronous; UI code should show loading or retain existing state while waiting.

```text
Widget -> repository -> Database.query/insert -> SQLite
       <- Future     <- result maps/count       <- file
```

# Syntax

```dart
final id = await db.insert('items', {'name': 'Pen'});
final rows = await db.query('items', where: 'id = ?', whereArgs: [id]);
```

# Complete Example 1

This runnable sample uses `sqflite` to create, insert, and read a record. Add dependencies `sqflite` and `path` in `pubspec.yaml`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openItemDb() async {
  final file = p.join(await getDatabasesPath(), 'sqflite_package.db');
  return openDatabase(
    file,
    version: 1,
    onCreate: (db, version) async {
      await db.execute(
        'CREATE TABLE items (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL)',
      );
    },
  );
}

void main() => runApp(const MaterialApp(home: ItemPage()));

class ItemPage extends StatefulWidget {
  const ItemPage({super.key});
  @override
  State<ItemPage> createState() => _ItemPageState();
}

class _ItemPageState extends State<ItemPage> {
  late final Future<Database> _db = openItemDb();
  String _result = 'Preparing database...';

  Future<void> _run() async {
    final db = await _db;
    final id = await db.insert('items', {'name': 'Pen'});
    final rows = await db.query('items', where: 'id = ?', whereArgs: [id]);
    if (mounted) {
      setState(() => _result = '${rows.single['name']} (row $id)');
    }
  }

  @override
  void initState() {
    super.initState();
    _run();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('sqflite package')),
        body: Center(child: Text(_result)),
        floatingActionButton: FloatingActionButton(
          onPressed: _run,
          child: const Icon(Icons.play_arrow),
        ),
      );
}
```

**Line-by-line guide:** Imports expose the widget framework, safe path utilities, and package database types. `openItemDb` creates one persistent filename in the platform's database directory; schema setup runs only on first creation. The table requires a name and generates an ID. `main` launches the example. The state caches a database future and display string. `_run` waits for the database, inserts a row, queries the generated ID using a placeholder, and updates UI only if mounted. `initState` runs the demonstration once. `build` shows the result and a button to repeat it. Widget tree: `MaterialApp -> ItemPage -> Scaffold -> AppBar + Center/Text + FAB`.

**Database flow:** `Dart insert helper -> SQLite row/id -> query helper -> maps -> state -> Text`.

**Execution:** Each button press inserts a new row and displays its ID. Removing `await` can cause the query to run before the insert completes; removing the table callback makes the first operation fail.

```text
+----------------------+
| sqflite package      |
|                      |
| Pen (row 1)          |
|                 [>]  |
+----------------------+
```

**Common mistakes:** Reopening the same file unnecessarily, using string concatenation for SQL, and assuming every `Database` call is synchronous.

**Professional use case:** A repository built on `sqflite` can isolate persistence details from screens and make tests easier.

# Complete Example 2

Update a row and use the count:

```dart
final count = await db.update(
  'items',
  {'name': 'Marker'},
  where: 'id = ?',
  whereArgs: [itemId],
);
```

The helper binds the map's values and returns affected rows. If `count` is zero, that ID did not match. Removing `where` changes every item's name.

# Complete Example 3

Delete a row safely:

```dart
final count = await db.delete(
  'items',
  where: 'id = ?',
  whereArgs: [itemId],
);
```

The predicate scopes the deletion. Confirm the intended row and refresh dependent UI after completion. Omitting `where` removes all records from the table.

# Advanced Example

Batch several operations to reduce overhead:

```dart
final batch = db.batch();
batch.insert('items', {'name': 'Pencil'});
batch.insert('items', {'name': 'Ruler'});
final results = await batch.commit(noResult: false);
```

A batch is useful for bulk work; use a transaction when atomic all-or-nothing semantics are required. Understand the package's batch/transaction behavior for the installed version.

# Under the Hood

The package maps Dart maps and arguments into SQLite statements. SQLite still performs SQL validation, constraint checking, query planning, and disk operations. A returned `List<Map<String, Object?>>` represents database result rows.

# Performance Notes

Reuse the database handle, select necessary columns, use transactions for dependent writes, and avoid blocking a frame with large synchronous data transformations after a query.

# Best Practices

- Pin compatible package versions through normal dependency management.
- Keep database opening centralized.
- Prefer helper methods for ordinary CRUD and raw SQL for queries the helpers cannot express clearly.
- Close a handle when its owning service is disposed, not after each operation.

# Common Mistakes

- Calling `openDatabase` from `build`.
- Forgetting to await a mutation before reading its result.
- Passing untrusted text into raw SQL.
- Ignoring migration callbacks.

# Debugging

Inspect package/platform support, database path, schema callbacks, and exact SQL errors. Verify the selected plugin version's API when upgrading.

# Interview Questions

1. **What does `sqflite` provide?** A Dart interface to SQLite on supported Flutter platforms.
2. **What does insert return?** Typically the inserted row ID.
3. **What does update return?** The number of affected rows.
4. **When use raw SQL?** For expressions, joins, or features not represented clearly by helper APIs.

# Try It Yourself

Add an update button that changes the inserted item and reports when no row matched.

# Challenge Project

Create a database service and repository around `sqflite`; build a small inventory UI without SQL inside widget methods.

# Quick Revision Sheet

- Package methods are asynchronous.
- `query` returns row maps; mutations return IDs/counts.
- Use placeholders, one long-lived handle, and schema migrations.
- Keep persistence code out of `build`.

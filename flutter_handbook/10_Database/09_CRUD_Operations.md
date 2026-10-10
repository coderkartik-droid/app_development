# Introduction

CRUD means Create, Read, Update, and Delete—the four basic operations on stored records. In SQLite, these correspond to `INSERT`, `SELECT`, `UPDATE`, and `DELETE`.

# Why this concept exists

Most database-backed screens allow users to add records, see them again, correct them, and remove them. A reliable CRUD layer defines these behaviors once and returns clear results to the UI.

# Real-world analogy

Managing a contact card: create a new card, read it, edit its phone number, or discard it.

# Internal Working

The repository converts Dart values to SQL operations. SQLite validates and applies a mutation, then the repository returns an ID or affected-row count. The app reloads or updates state after the operation completes.

```text
Create: Dart map -> INSERT -> row ID
Read:   SELECT -> result maps
Update: map + predicate -> UPDATE -> count
Delete: predicate -> DELETE -> count
```

# Syntax

```dart
await db.insert('tasks', {'title': 'Write tests'});
final rows = await db.query('tasks');
await db.update('tasks', {'title': 'Test'}, where: 'id = ?', whereArgs: [id]);
await db.delete('tasks', where: 'id = ?', whereArgs: [id]);
```

# Complete Example 1

This runnable app implements create, read, and delete for a task. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openCrudDb() async {
  final file = p.join(await getDatabasesPath(), 'crud_demo.db');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE tasks (id INTEGER PRIMARY KEY AUTOINCREMENT, title TEXT NOT NULL)',
    );
  });
}

void main() => runApp(const MaterialApp(home: CrudPage()));

class CrudPage extends StatefulWidget {
  const CrudPage({super.key});
  @override
  State<CrudPage> createState() => _CrudPageState();
}

class _CrudPageState extends State<CrudPage> {
  late final Future<Database> _db = openCrudDb();
  List<Map<String, Object?>> _rows = [];

  Future<void> _refresh() async {
    final rows = await (await _db).query('tasks', orderBy: 'id DESC');
    if (mounted) setState(() => _rows = rows);
  }

  Future<void> _create() async {
    await (await _db).insert('tasks', {'title': 'CRUD task'});
    await _refresh();
  }

  Future<void> _delete(int id) async {
    await (await _db).delete('tasks', where: 'id = ?', whereArgs: [id]);
    await _refresh();
  }

  @override
  void initState() {
    super.initState();
    _refresh();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Task CRUD')),
        body: ListView(
          children: _rows.map((row) {
            final id = row['id']! as int;
            return ListTile(
              title: Text(row['title']! as String),
              trailing: IconButton(
                icon: const Icon(Icons.delete),
                onPressed: () => _delete(id),
              ),
            );
          }).toList(),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: _create,
          child: const Icon(Icons.add),
        ),
      );
}
```

**Line-by-line guide:** Imports provide UI, path, and SQLite. `openCrudDb` creates a stable database with an auto-ID task table. The state caches the database future and result rows. `_refresh` reads newest-first and sets state if mounted. `_create` inserts a task then refreshes. `_delete` removes only the selected ID using a bound argument, then reloads. `initState` obtains initial rows. `build` creates one tile per row, with a delete button, and a FAB for creation. Widget tree: `MaterialApp -> CrudPage -> Scaffold -> AppBar + ListView/ListTile/IconButton + FAB`.

**Database flow:** `tap add/delete -> INSERT/DELETE -> SQLite -> SELECT -> setState -> rebuild`.

**Execution:** Launch loads saved tasks. Add inserts; delete removes the selected row. Without the delete predicate, one tap clears the entire table; without refresh, the data is correct on disk but the list is stale.

```text
+----------------------+
| Task CRUD            |
|----------------------|
| CRUD task        [x] |
|                 [+]  |
+----------------------+
```

**Common mistakes:** Forgetting a predicate, trusting map casts without schema awareness, and not refreshing state after successful writes.

**Professional use case:** Repository CRUD methods back list/detail/edit screens while centralizing data rules.

# Complete Example 2

Update one row and verify the count:

```dart
final changed = await db.update(
  'tasks',
  {'title': 'CRUD task reviewed'},
  where: 'id = ?',
  whereArgs: [taskId],
);
if (changed == 0) {
  // The record may have been deleted or the ID may be stale.
}
```

The map contains modified fields, and the predicate scopes the update. A zero result is a useful domain outcome, not necessarily an engine failure.

# Complete Example 3

Read with a projection and ordering:

```dart
final titles = await db.query(
  'tasks',
  columns: ['id', 'title'],
  orderBy: 'title COLLATE NOCASE ASC',
);
```

The projection avoids unneeded columns and the order is performed by SQLite. It does not mutate stored data.

# Advanced Example

Make a multi-step create atomic:

```dart
await db.transaction((txn) async {
  final taskId = await txn.insert('tasks', {'title': 'Ship release'});
  await txn.insert('task_events', {'task_id': taskId, 'event': 'created'});
});
```

If either operation throws, the transaction rolls back. Use `txn` within its callback and create a foreign key from `task_events.task_id` to `tasks.id`.

# Under the Hood

SQLite executes each statement and returns result rows or an affected count. `sqflite` represents query output as maps; the application is responsible for type-safe mapping and state refresh.

# Performance Notes

Paginate or limit large lists; index frequently filtered/sorted columns. Prefer a batch/transaction for bulk changes rather than many unrelated round trips.

# Best Practices

- Always scope update/delete operations.
- Validate user input and enforce critical constraints in schema.
- Return affected counts and handle zero-row outcomes.
- Keep CRUD APIs typed at repository boundaries.

# Common Mistakes

- Omitting `where` on destructive operations.
- Updating the entire UI before persistence finishes.
- Loading every row for a count/filter.
- Treating an empty list and a database error as the same state.

# Debugging

Check the SQL predicate, bound values, result count, and table contents. Add tests for missing IDs and duplicate/invalid data.

# Interview Questions

1. **What does CRUD stand for?** Create, Read, Update, Delete.
2. **What does an update count of zero mean?** No row matched the predicate.
3. **Why use a transaction for related records?** To prevent partial success.
4. **What is a projection?** Selecting only needed columns.

# Try It Yourself

Add an edit dialog and persist an updated task title. Handle blank titles and stale IDs.

# Challenge Project

Implement a contacts database with full CRUD, search, empty/error/loading states, and tests for repository methods.

# Quick Revision Sheet

- Create: `insert`; Read: `query`/`rawQuery`.
- Update and delete must have intentional predicates.
- Refresh or reconcile UI state after writes.
- Use transactions for related writes.

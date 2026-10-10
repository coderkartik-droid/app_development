# Introduction

SQLite CRUD uses SQL statements to insert, select, update, and delete rows. Flutter applications call these statements through a database package such as `sqflite`.

# Why this concept exists

Relational local data needs precise record identity, filtering, and consistency. SQL provides those operations directly.

# Real-world analogy

CRUD is a library catalogue: add a book, search for it, revise its record, and remove it with a precise catalogue number.

# Internal Working

The driver sends SQL and bound parameters to SQLite. SQLite executes the statement, applies constraints, and returns rows or an affected-row count. Transactions group operations.

# Syntax

```sql
INSERT INTO tasks (title, done) VALUES (?, ?);
SELECT id, title, done FROM tasks WHERE id = ?;
UPDATE tasks SET done = ? WHERE id = ?;
DELETE FROM tasks WHERE id = ?;
```

# Complete Example 1

Below is a CRUD repository outline for a `sqflite`-style API. Add a compatible SQLite plugin and platform setup; exact return types can vary by package version.

```dart
import 'package:sqflite/sqflite.dart';

class TaskRepository {
  TaskRepository(this.database);
  final Database database;

  Future<int> create(String title) {
    return database.insert(
      'tasks',
      {'title': title, 'done': 0},
      conflictAlgorithm: ConflictAlgorithm.abort,
    );
  }

  Future<List<Map<String, Object?>>> readAll() {
    return database.query(
      'tasks',
      columns: ['id', 'title', 'done'],
      orderBy: 'id DESC',
    );
  }

  Future<int> updateDone(int id, bool done) {
    return database.update(
      'tasks',
      {'done': done ? 1 : 0},
      where: 'id = ?',
      whereArgs: [id],
    );
  }

  Future<int> delete(int id) {
    return database.delete(
      'tasks',
      where: 'id = ?',
      whereArgs: [id],
    );
  }
}
```

### Line-by-line explanation

- The repository receives an already-open database, so database lifecycle is managed elsewhere.
- `create` inserts a title and integer-backed completion flag.
- `ConflictAlgorithm.abort` rejects conflicting insert behavior rather than silently overwriting.
- `readAll` selects only required columns and orders newest ID first.
- `updateDone` uses a `WHERE id = ?` predicate and binds the ID through `whereArgs`.
- `delete` similarly targets a single row.
- The integer representation for booleans is explicit; mapping may be centralized in a model converter.

### Widget tree

The repository has no widget tree; UI calls its methods and displays returned records.

### Execution flow

```text
Widget action → repository method → parameterized database call
                                     |
                         rows/count returned
                                     v
                              UI state refresh
```

### ASCII phone preview

```text
+---------------------------+
|            Tasks          |
+---------------------------+
| □ Send invoice       [x]  |
| ✓ Book dentist      [x]  |
|                           |
+---------------------------+
```

### Example explanation

Every user-supplied value is bound, not interpolated into SQL. The update and delete methods target one primary key. Their integer results let the caller detect a missing row.

If `where` is omitted from update/delete, every row may be affected. If `whereArgs` is omitted while the query contains a placeholder, the statement cannot bind correctly.

**Common mistakes:** Do not return raw database maps throughout a large app; map rows to typed domain models at the repository boundary.

**Professional use case:** Offline tasks, notes, or inventory requiring filtered queries and durable updates.

# Complete Example 2

Wrap a multi-step create in a transaction:

```dart
await database.transaction((txn) async {
  final taskId = await txn.insert('tasks', taskValues);
  await txn.insert('audit', {'taskId': taskId, 'action': 'created'});
});
```

Both operations succeed together or roll back according to transaction semantics.

# Complete Example 3

Search using a bound pattern:

```dart
final rows = await database.query(
  'tasks',
  where: 'title LIKE ?',
  whereArgs: ['%$query%'],
);
```

The `%` wildcards are part of the bound parameter value; the user's input is not interpreted as SQL syntax.

# Advanced Example

Use repository-level methods that return typed objects, map database failures to app-level errors, and expose pagination. Ensure database initialization is performed once, migrations are transactional where supported, and tests create isolated temporary databases.

# Under the Hood

SQLite parses SQL, binds parameters, chooses a query plan, and applies operations to database pages. A transaction uses the configured journal mechanism to maintain consistency. The Flutter plugin moves operations through a platform/native boundary.

# Performance Notes

- Add indexes based on measured query patterns.
- Select specific columns and limit results.
- Batch inserts in a transaction.
- Avoid one query per row (N+1).
- Close database resources at the owning application/service boundary.

# Best Practices

- Always use parameter binding for values.
- Use primary keys and constraints.
- Check affected rows when needed.
- Map database rows to typed models.
- Test migrations, rollback, and empty results.

# Common Mistakes

- Missing `WHERE` in update/delete.
- Concatenating SQL values.
- Using `SELECT *` for every screen.
- Ignoring transaction failure.
- Sharing a database handle with unclear lifecycle.

# Debugging

- Log query structure and execution duration, not sensitive bound values.
- Inspect schema and query plan.
- Check affected-row counts.
- Test with empty database and legacy database versions.
- Reproduce transaction rollback behavior.

# Interview Questions

1. Why bind parameters? To keep values separate from SQL syntax and prevent injection.
2. What does update's affected-row count tell you? How many rows matched and changed.
3. Why wrap related CRUD operations in a transaction? To preserve all-or-nothing consistency.
4. Why map row maps to models? To isolate schema representation from application logic.

# Try It Yourself

1. Add a title update method and validate its result count.
2. Add a due date column and migration.
3. Add a search index if the query pattern benefits.
4. Test deleting a nonexistent ID.

# Challenge Project

Build a SQLite task repository with typed models, CRUD, search, pagination, transaction-based audit rows, and a migration test.

# Quick Revision Sheet

- CRUD: `insert`, `query`, `update`, `delete`.
- Bind values with placeholders/whereArgs.
- Always target rows with precise predicates.
- Use transactions for related operations.
- Return typed domain models and test migrations.

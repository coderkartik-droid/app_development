# Database Cheat Sheet

## Most Important Concepts

- SQLite is an embedded relational engine; `sqflite` is a Flutter integration.
- Schema describes tables, columns, keys, constraints, and indexes.
- Database futures are asynchronous; UI state updates separately.
- Existing databases need versioned migrations.

## Syntax Reference

```sql
CREATE TABLE tasks (
  id INTEGER PRIMARY KEY,
  title TEXT NOT NULL,
  done INTEGER NOT NULL DEFAULT 0
);
CREATE INDEX idx_tasks_done ON tasks(done);
SELECT id, title FROM tasks WHERE done = ? ORDER BY id LIMIT ?;
```

## CRUD / Frequently Used Methods

| Operation | `sqflite` |
|---|---|
| Create | `db.insert(table, values)` |
| Read | `db.query(table, where, whereArgs)` |
| Update | `db.update(table, values, where, whereArgs)` |
| Delete | `db.delete(table, where, whereArgs)` |
| SQL | `db.execute(...)`, `db.rawQuery(...)` |
| Atomic group | `db.transaction((txn) async { ... })` |

## Parameters & Properties

- `version`: schema version.
- `onCreate`: new database schema creation.
- `onUpgrade`: existing schema transition.
- `whereArgs`: safely bind query values.
- `columns`: projection; `orderBy`, `limit`: result shape.
- Foreign keys must be enabled/configured where required.

## Short Example

```dart
final rows = await db.query(
  'tasks',
  columns: ['id', 'title'],
  where: 'done = ?',
  whereArgs: [0],
);
```

## Common Mistakes

- Missing `where` on update/delete.
- Concatenating user input into SQL.
- Assuming `onCreate` updates existing installs.
- Calling database methods from `build`.
- Indexing every column without measuring.

## Interview Revision

- Primary key uniquely identifies a row; foreign key references a row.
- One-to-many FK is on the many side; many-to-many uses a junction table.
- Transaction groups writes atomically.
- Index improves selected reads but costs storage and write time.
- Migration evolves schema while preserving stored data.

## Best Practices

- Use stable file paths, constraints, typed repositories, and bound parameters.
- Test fresh creation, old-version upgrades, rollback, and empty results.
- Keep transactions short; use `txn` within callbacks.
- Measure with realistic data and `EXPLAIN QUERY PLAN`.

## Memory Tricks

- **CRUD:** insert, query, update, delete.
- **FK on many side.**
- **Multi-write invariant = transaction.**
- **Released schema change = migration.**

## One-Page Summary

Design explicit tables and constraints, bind values, scope mutations, transact dependent writes, and migrate versioned schemas. Filter, project, and limit in SQL; add indexes only for measured access paths. Refresh app state after database changes.

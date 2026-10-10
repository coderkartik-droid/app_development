# Local Storage Cheat Sheet

## Most Important Concepts

- Choose storage by data shape, query needs, sensitivity, and volume.
- Preferences are for small settings; object/relational databases are for records.
- Credentials/secrets need platform-secure handling, not ordinary database columns.
- Local data does not automatically sync, back up, or encrypt itself.

## Quick Comparison

| Option | Best for | Query/relationship strength |
|---|---|---|
| Preferences | Small primitive settings | Very limited |
| Hive/object store | Object/key-value access | Model/query dependent |
| SQLite (`sqflite`, Drift) | Structured relational data | SQL, joins, transactions |
| Secure storage | Small secrets/keys | Not a general database |

## Syntax Reference

```dart
// SQLite via sqflite
await db.insert('notes', {'title': title});
final rows = await db.query('notes', where: 'id = ?', whereArgs: [id]);
```

## Common Classes / Methods

- SQLite: `openDatabase`, `query`, `insert`, `update`, `delete`, `transaction`
- Drift: generated tables/DAOs, `get()`, `watch()`
- Hive/object stores: open collection/box, read/write by key
- Database lifecycle: schema version, create, upgrade, close

## Parameters & Properties

- Database `path` and stable filename identify the file.
- `version` selects schema lifecycle.
- `where` scopes mutation; `whereArgs` binds values.
- `columns`, `orderBy`, `limit` shape result size.
- Transaction callback groups atomic operations.

## Short Example

```dart
await db.transaction((txn) async {
  final id = await txn.insert('orders', orderValues);
  await txn.insert('order_items', {'order_id': id, ...itemValues});
});
```

## Common Mistakes

- Running I/O in `build`.
- Omitting a `where` clause on update/delete.
- Editing only `onCreate` instead of writing migrations.
- Storing money in floating point.
- Assuming local database data is encrypted or synchronized.

## Interview Revision

- Primary key identifies; foreign key references.
- One-to-many foreign key lives on the many side.
- Many-to-many uses a junction table.
- Index trades storage/write work for selected read speed.
- Transaction provides atomicity for related writes.

## Best Practices

- Use parameterized values, constraints, and explicit keys.
- Keep SQL behind a repository/DAO.
- Test new installs and every supported migration path.
- Measure query plans before adding indexes.

## Memory Tricks

- CRUD = Create, Read, Update, Delete.
- Foreign key is on the “many” side.
- Related writes? Transaction.
- Schema changed? Migration.

## One-Page Summary

Choose a store that matches the data and queries. For relational records, use schema constraints, bound SQL values, scoped mutations, transactions, indexes based on measurement, and versioned migrations. Keep UI state separate from stored data.

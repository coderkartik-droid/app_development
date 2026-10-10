# Introduction

Database best practices are habits that preserve data integrity, user data, and responsive behavior as an app grows. The right schema and lifecycle decisions are easier to make before many releases depend on them.

# Why this concept exists

Database errors can silently corrupt state or permanently erase records. Consistent practices reduce risk and make performance and upgrades predictable.

# Real-world analogy

Database engineering is like maintaining a bridge: strong materials matter, but inspections, load limits, and safe repair procedures matter too.

# Internal Working

A robust persistence design connects a versioned schema, typed repository, parameterized operations, transactions, and UI states. Tests verify both fresh setup and upgrades.

```text
Schema/constraints -> repository -> transaction/query -> typed result -> state
       ^                   tests + migration verification              |
       +----------------------------- feedback -------------------------+
```

# Syntax

```dart
await db.transaction((txn) async {
  await txn.insert('parent', parentValues);
  await txn.insert('child', childValues);
});
```

# Complete Example 1

This runnable sample shows parameterized insertion and lookup with an explicit constraint. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openBestPracticeDb() async {
  final file = p.join(await getDatabasesPath(), 'best_practice_demo.db');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE labels (id INTEGER PRIMARY KEY, value TEXT NOT NULL UNIQUE)',
    );
  });
}

void main() => runApp(const MaterialApp(home: BestPracticePage()));

class BestPracticePage extends StatefulWidget {
  const BestPracticePage({super.key});
  @override
  State<BestPracticePage> createState() => _BestPracticePageState();
}

class _BestPracticePageState extends State<BestPracticePage> {
  late final Future<Database> _db = openBestPracticeDb();
  String _result = 'No row yet';

  Future<void> _save() async {
    final db = await _db;
    await db.insert('labels', {'id': 1, 'value': 'Bound value'});
    final rows = await db.query(
      'labels',
      columns: ['value'],
      where: 'id = ?',
      whereArgs: [1],
    );
    if (mounted) {
      setState(() => _result = rows.isEmpty ? 'Missing' : rows.single['value']! as String);
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Safe database pattern')),
        body: Center(child: Text(_result)),
        floatingActionButton: FloatingActionButton(
          onPressed: _save,
          child: const Icon(Icons.save),
        ),
      );
}
```

**Line-by-line guide:** Imports provide the UI, path, and SQLite. `openBestPracticeDb` uses a stable file and creates a table with a unique required value. The page caches an open future. `_save` inserts a row, selects a projection by a bound ID, handles an empty result, and updates state only while mounted. The button triggers the operation. Widget tree: `MaterialApp -> BestPracticePage -> Scaffold -> AppBar + Center/Text + FAB`.

**Database flow:** `insert -> uniqueness constraint -> parameterized select -> typed display result`.

**Execution:** First press inserts and displays a value; repeating it encounters the unique ID unless an intentional upsert is chosen. That visible error is preferable to silently inventing success. Removing the uniqueness constraint permits duplicates; using string interpolation would be unsafe.

```text
+----------------------+
| Safe database pattern|
|                      |
| Bound value          |
|                 [S]  |
+----------------------+
```

**Common mistakes:** Treating conflict handling as incidental and masking constraint failures.

**Professional use case:** Repository operations make database invariants explicit and testable.

# Complete Example 2

Use parameterized values and explicit columns:

```dart
final rows = await db.query(
  'products',
  columns: ['id', 'name'],
  where: 'category_id = ? AND archived = ?',
  whereArgs: [categoryId, 0],
);
```

This narrows results and safely binds both values. Keep table/column identifiers fixed; placeholders bind values, not SQL structure.

# Complete Example 3

Version every schema change. Test fresh install, each supported old version, invalid data, empty data, and rollback. Keep a copy of fixtures to reproduce failures safely.

# Advanced Example

Define data retention, backup/export, sensitive-field handling, and deletion semantics with product requirements. A local database is not automatically encrypted, backed up, synchronized, or protected from a compromised device.

# Under the Hood

Constraints and transactions enforce rules at the engine boundary; repository types make expected values clearer in Dart. Migration callbacks transform schema and stored data under version control.

# Performance Notes

Measure real query plans, limit histories, index frequent access paths, and avoid doing database work in `build`.

# Best Practices

- Use stable file paths and one clear database owner.
- Add constraints, foreign keys, and indexes intentionally.
- Bind values and scope mutations.
- Use transactions for invariant-related writes.
- Test migrations and keep user data safe.
- Distinguish loading, empty, failure, and success UI states.

# Common Mistakes

- Assuming local data has automatic backup or sync.
- Storing monetary values as floating point.
- Logging entire rows.
- Adding indexes speculatively.
- Recreating a database to avoid migration work.

# Debugging

Trace path, schema version, statement, arguments, result count, mapping, and state refresh. Inspect a copy of the actual database when needed.

# Interview Questions

1. **Why define constraints in SQL?** They protect invariants for every writer.
2. **Why avoid SQL interpolation?** It risks injection and escaping errors.
3. **What belongs in a transaction?** Mutations that collectively preserve a business invariant.
4. **What should a production migration test cover?** Old schema, representative data, transformed result, and failure behavior.

# Try It Yourself

Audit a database feature for path stability, constraints, parameterization, transactions, migration coverage, logging, and result UI.

# Challenge Project

Implement a local catalog with typed repository methods, schema versioning, search indexes, temporary-database tests, and an explicit retention policy.

# Quick Revision Sheet

- Data integrity begins in schema constraints.
- Bind values, scope writes, and transact related changes.
- Measure before indexing.
- Test upgrades with real old schemas.
- Local database behavior does not imply backup, encryption, or synchronization.

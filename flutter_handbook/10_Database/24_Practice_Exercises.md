# Introduction

These exercises build database skill through progressively harder modeling, querying, mutation, and migration tasks. Use a real local SQLite database for integration-level practice.

# Why this concept exists

Database knowledge becomes practical only when you can design a schema, express queries, handle errors, and prove that data survives upgrades.

# Real-world analogy

Practice is a workshop: begin with one well-fitted part, then assemble components into a reliable machine.

# Internal Working

For each exercise, define the invariant, schema, operation, expected result, and failure case before writing UI code.

```text
Requirement -> tables/constraints -> repository operation -> test -> UI state
```

# Syntax

```sql
SELECT category_id, COUNT(*) AS total
FROM expenses
GROUP BY category_id;
```

# Complete Example 1

This runnable sample adds an expense and displays a SQL-computed count. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openPracticeDb() async {
  final file = p.join(await getDatabasesPath(), 'practice_demo.db');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE expenses (id INTEGER PRIMARY KEY AUTOINCREMENT, label TEXT NOT NULL)',
    );
  });
}

void main() => runApp(const MaterialApp(home: PracticePage()));

class PracticePage extends StatefulWidget {
  const PracticePage({super.key});
  @override
  State<PracticePage> createState() => _PracticePageState();
}

class _PracticePageState extends State<PracticePage> {
  late final Future<Database> _db = openPracticeDb();
  int _count = 0;

  Future<void> _refresh() async {
    final value = Sqflite.firstIntValue(
          await (await _db).rawQuery('SELECT COUNT(*) FROM expenses'),
        ) ??
        0;
    if (mounted) setState(() => _count = value);
  }

  Future<void> _add() async {
    await (await _db).insert('expenses', {'label': 'Practice entry'});
    await _refresh();
  }

  @override
  void initState() {
    super.initState();
    _refresh();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Database exercise')),
        body: Center(child: Text('Expense count: $_count')),
        floatingActionButton: FloatingActionButton(
          onPressed: _add,
          child: const Icon(Icons.add),
        ),
      );
}
```

**Line-by-line guide:** Imports provide Flutter, path, and SQLite. `openPracticeDb` opens a stable file and creates an expense table. `_refresh` runs an aggregate count in SQLite, converts it to an integer, and updates state while mounted. `_add` inserts a record and refreshes. `initState` loads the count on launch. `build` shows the count and an add button. Widget tree: `MaterialApp -> PracticePage -> Scaffold -> AppBar + Center/Text + FAB`.

**Database flow:** `tap -> INSERT -> COUNT(*) -> state -> visible count`.

**Execution:** Each tap increments the displayed count and the value survives restarts. Removing the refresh leaves the old count visible; counting rows in Dart instead would unnecessarily load all labels.

```text
+----------------------+
| Database exercise    |
|                      |
| Expense count: 3     |
|                 [+]  |
+----------------------+
```

**Common mistakes:** Testing only the happy path and failing to reset fixtures between tests.

**Professional use case:** A small exercise grows into a repository test suite and a production-backed feature.

# Complete Example 2

**Exercise: CRUD.** Create a `books` table with title and publication year. Implement add, lookup-by-ID, update-title, and delete-by-ID. Test no-match counts and duplicate keys. Do not use `SELECT *` when the screen needs only a title.

# Complete Example 3

**Exercise: relationships.** Model authors and books as one-to-many. Add a foreign key, choose delete behavior, join book titles to author names, and explain what happens when an author has no books.

# Advanced Example

**Exercise: migration.** Start from a v1 schema with expense category text. In v2, create a category table, migrate distinct categories, add a category foreign key, and verify every old expense remains associated. Test fresh v2 creation and v1-to-v2 upgrade.

# Under the Hood

Exercise tests should assert persisted database state, not only widget text. Use a temporary database per test and close it during teardown.

# Performance Notes

Use realistic record counts in query exercises. Compare SQL filtering/aggregation against loading all rows into Dart.

# Best Practices

- State invariants before writing SQL.
- Seed deterministic fixtures.
- Test success, empty, invalid, and rollback cases.
- Keep test databases isolated.

# Common Mistakes

- Sharing a persistent test database between cases.
- Ignoring foreign-key setup.
- Asserting implementation details instead of observable database behavior.
- Skipping migration tests.

# Debugging

Print safe counts and query plans, inspect the temporary schema, and reduce a failing test to the smallest reproducible operation.

# Interview Questions

1. **What is the value of integration tests?** They verify package/engine behavior and schema together.
2. **Why isolate test databases?** To prevent order-dependent failures and data leakage.
3. **What should a migration test assert?** New schema plus preserved/transformed prior data.
4. **Why seed fixed fixture data?** Deterministic queries and expectations.

# Try It Yourself

Complete exercises 2 and 3 without building a UI; test repository methods first.

# Challenge Project

Choose one exercise, create a Flutter UI, a repository, a database schema, tests for normal and failure paths, and a written migration plan.

# Quick Revision Sheet

- Model invariants first.
- Test CRUD, relationships, transactions, and migrations.
- Isolate temporary databases.
- Assert stored results, not only rendered labels.

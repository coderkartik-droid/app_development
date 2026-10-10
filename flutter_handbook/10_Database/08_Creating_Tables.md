# Introduction

A table defines the shape of one kind of record. It lists columns, their SQLite types, constraints, and usually a primary key. Thoughtful table design makes invalid data harder to store and expected queries easier to write.

# Why this concept exists

Without a schema, records can drift into inconsistent shapes. A table contract tells SQLite what values are allowed and how each row is identified.

# Real-world analogy

A table is a standardized form: every submitted form has the same labeled fields, and required fields cannot be left blank.

# Internal Working

`CREATE TABLE` stores a schema definition in SQLite metadata. On inserts and updates, SQLite checks types and constraints. The `sqflite` call runs asynchronously and completes when the engine has applied the statement.

```text
CREATE TABLE -> schema metadata
INSERT map -> column mapping -> constraint checks -> stored row
```

# Syntax

```sql
CREATE TABLE tasks (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT NOT NULL,
  is_done INTEGER NOT NULL DEFAULT 0
);
```

# Complete Example 1

This app creates a constrained table and inserts a row. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openTaskDatabase() async {
  final file = p.join(await getDatabasesPath(), 'tables_demo.db');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute('''
      CREATE TABLE tasks (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        is_done INTEGER NOT NULL DEFAULT 0
      )
    ''');
  });
}

void main() => runApp(const MaterialApp(home: TablePage()));

class TablePage extends StatefulWidget {
  const TablePage({super.key});
  @override
  State<TablePage> createState() => _TablePageState();
}

class _TablePageState extends State<TablePage> {
  late final Future<Database> _db = openTaskDatabase();
  String _status = 'Creating table...';

  Future<void> _createAndInsert() async {
    final db = await _db;
    final id = await db.insert('tasks', {'title': 'Review table schema'});
    final row = (await db.query('tasks', where: 'id = ?', whereArgs: [id])).single;
    if (mounted) setState(() => _status = '${row['title']} (done=${row['is_done']})');
  }

  @override
  void initState() {
    super.initState();
    _createAndInsert();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Create a table')),
        body: Center(child: Text(_status)),
      );
}
```

**Line-by-line guide:** The imports load Flutter, path joining, and SQLite. `openTaskDatabase` chooses a stable database file. The multi-line SQL defines an auto-generated key, required title, and integer completion flag defaulting to zero. The state holds one open future and a status string. `_createAndInsert` inserts a title, then queries the generated ID and displays both title and default flag. `initState` runs it once. `build` shows the result. Widget tree: `MaterialApp -> TablePage -> Scaffold -> AppBar + Center/Text`.

**Database flow:** `CREATE TABLE schema -> INSERT title -> default is_done=0 -> SELECT by ID -> UI`.

**Execution:** On first run the schema is created; insertion omits `is_done` so SQLite supplies the default. Removing `NOT NULL` allows incomplete titles; removing the default requires callers to supply the flag.

```text
+----------------------+
| Create a table       |
|                      |
| Review table schema  |
| (done=0)             |
+----------------------+
```

**Common mistakes:** Changing SQL without incrementing schema version, using inconsistent column names, and using a nullable column for required data.

**Professional use case:** A tasks table with explicit invariants ensures every saved task can be listed and interpreted consistently.

# Complete Example 2

Declare a unique business key:

```sql
CREATE TABLE categories (
  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL UNIQUE
);
```

`UNIQUE` rejects duplicate category names (subject to SQLite's collation and null rules). Enforce uniqueness in the database, not only by checking the UI; two writes can race.

# Complete Example 3

Create an index for a common query:

```sql
CREATE INDEX idx_tasks_done ON tasks(is_done);
```

The index can make filtering by completion faster for sufficiently useful workloads, but consumes space and adds write work. Remove it if no query benefits.

# Advanced Example

Use a check constraint for an allowed range:

```sql
CREATE TABLE ratings (
  id INTEGER PRIMARY KEY,
  score INTEGER NOT NULL CHECK (score BETWEEN 1 AND 5)
);
```

The engine enforces the range for every write path, including future screens or imports.

# Under the Hood

SQLite records table and index definitions in its schema catalog. Constraints are evaluated as part of mutations. Column affinity influences storage but does not mean every SQL type is a strict Dart type.

# Performance Notes

Avoid adding speculative indexes. Index foreign-key columns used for joins/deletes and inspect plans on realistic data.

# Best Practices

- Choose clear, stable table and column names.
- Add primary keys and required-field constraints.
- Represent booleans consistently as 0/1.
- Change schemas only through numbered migrations.

# Common Mistakes

- Assuming `onCreate` applies revised SQL to old installs.
- Omitting constraints and relying solely on form validation.
- Using reserved SQL words as identifiers without quoting.
- Creating an index on every column.

# Debugging

Use `PRAGMA table_info(tasks)` and `sqlite_master` to inspect live schema. Confirm the database file/version matches the app build.

# Interview Questions

1. **What is a primary key?** A column or column set that uniquely identifies rows.
2. **What does `NOT NULL` enforce?** The column must receive a non-null value.
3. **Why put constraints in the database?** They protect integrity across all writers.
4. **What is an index cost?** Extra storage and additional maintenance during writes.

# Try It Yourself

Add a due date and a check constraint for completion values. Try to insert an invalid completion value and observe the error.

# Challenge Project

Design a schema for products, categories, and stock adjustments. Draw the relationships and list constraints before creating tables.

# Quick Revision Sheet

- A schema defines columns and invariants.
- Primary keys identify rows; constraints reject invalid data.
- Indexes help selected queries and cost storage/write time.
- Version schema changes through migrations.

# Introduction

A migration changes an existing database schema while preserving the records already stored. Examples include adding a column, creating an index, or splitting one table into related tables.

# Why this concept exists

After an app is released, users may upgrade without reinstalling. Their database has an older schema, so a new app version must transform it safely rather than assuming a clean install.

# Real-world analogy

A migration is a renovation plan for an occupied building: add a room or reroute plumbing while keeping residents and their belongings safe.

# Internal Working

The database stores a user/schema version. On open, the package compares the stored version with the requested version and runs the upgrade callback. Each step should advance one version and execute in a transaction.

```text
installed schema v1 -> app requests v2 -> migration v1->v2 -> current schema v2
```

# Syntax

```dart
onUpgrade: (db, oldVersion, newVersion) async {
  if (oldVersion < 2) {
    await db.execute('ALTER TABLE notes ADD COLUMN pinned INTEGER NOT NULL DEFAULT 0');
  }
}
```

# Complete Example 1

This example defines a versioned SQLite opener. On an existing version 1 database it adds a column; on a fresh install `onCreate` builds the latest schema. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openMigratingDb() async {
  final file = p.join(await getDatabasesPath(), 'migration_demo.db');
  return openDatabase(
    file,
    version: 2,
    onCreate: (db, version) async {
      await db.execute('''
        CREATE TABLE notes (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          title TEXT NOT NULL,
          pinned INTEGER NOT NULL DEFAULT 0
        )
      ''');
    },
    onUpgrade: (db, oldVersion, newVersion) async {
      if (oldVersion < 2) {
        await db.execute(
          'ALTER TABLE notes ADD COLUMN pinned INTEGER NOT NULL DEFAULT 0',
        );
      }
    },
  );
}

void main() => runApp(const MaterialApp(home: MigrationPage()));

class MigrationPage extends StatefulWidget {
  const MigrationPage({super.key});
  @override
  State<MigrationPage> createState() => _MigrationPageState();
}

class _MigrationPageState extends State<MigrationPage> {
  late final Future<Database> _db = openMigratingDb();
  String _status = 'Opening schema version 2...';

  Future<void> _check() async {
    try {
      final db = await _db;
      final columns = await db.rawQuery('PRAGMA table_info(notes)');
      if (mounted) setState(() => _status = 'Ready; ${columns.length} columns');
    } catch (error) {
      if (mounted) setState(() => _status = 'Migration failed: $error');
    }
  }

  @override
  void initState() {
    super.initState();
    _check();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Database migration')),
        body: Center(child: Text(_status)),
      );
}
```

**Line-by-line guide:** Imports provide UI, path handling, and `sqflite`. `openMigratingDb` opens a stable file and requests schema version 2. A fresh install creates the complete latest table. Existing databases below version 2 add the `pinned` column with a non-null default so old rows remain valid. The state stores one open future and status. `_check` awaits opening, inspects columns with `PRAGMA`, and reports errors instead of hiding them. `initState` invokes it once. Widget tree: `MaterialApp -> MigrationPage -> Scaffold -> AppBar + Center/Text`.

**Database flow:** `open version=2 -> compare stored version -> onCreate OR onUpgrade -> schema check -> UI`.

**Execution:** New installs get both columns; a version 1 database is upgraded and existing notes receive `pinned=0`. Removing the version increment prevents `onUpgrade` from running; removing the default may make the `ALTER TABLE` fail for existing rows.

```text
+----------------------+
| Database migration  |
|                      |
| Ready; 3 columns     |
+----------------------+
```

**Common mistakes:** Editing `onCreate` and expecting existing installs to change, skipping intermediate versions, and destructive table recreation.

**Professional use case:** A released notes app adds pinning without losing a user's existing notes.

# Complete Example 2

A sequential migration supports users upgrading across multiple releases:

```dart
if (oldVersion < 2) {
  await db.execute('ALTER TABLE notes ADD COLUMN pinned INTEGER NOT NULL DEFAULT 0');
}
if (oldVersion < 3) {
  await db.execute('CREATE INDEX idx_notes_pinned ON notes(pinned)');
}
```

Version 1 runs both steps; version 2 runs only the second. Keep each migration in source control. If an operation is not idempotent, rely on the version progression and test every supported starting version.

# Complete Example 3

For a column rename, a portable strategy may create a replacement table, copy and transform rows, drop the old table, then rename the new table—all within the migration transaction. Plan constraints and indexes too. Never drop the original before verifying the copy succeeds.

# Advanced Example

Test migrations using real schema snapshots from every supported old version. Populate representative data, upgrade, and assert both schema and values. A migration should be deterministic and preserve user data; export/backup policy matters for especially valuable records.

# Under the Hood

SQLite's schema version is compared by the wrapper when opening. Upgrade callbacks are generally run transactionally by the package; confirm installed package semantics. A thrown migration error should prevent marking the database as upgraded.

# Performance Notes

Adding an index to a large table takes time. Keep migrations short, avoid loading the entire database into Dart, and consider user-visible startup impact.

# Best Practices

- Increment schema version for every released schema change.
- Write sequential, data-preserving migrations.
- Test fresh creation and every supported upgrade path.
- Do not delete user data to avoid handling a migration.

# Common Mistakes

- Assuming `onCreate` migrates old files.
- Setting version to a value that skips needed steps.
- Renaming/dropping columns without preserving data.
- Failing to test an upgrade on an actual old schema.

# Debugging

Inspect `PRAGMA user_version`, `PRAGMA table_info(table)`, and migration logs. Reproduce with a copy of the old database; don't test destructive migrations on user data.

# Interview Questions

1. **What is a database migration?** A versioned transformation of existing schema/data.
2. **Why does `onCreate` not suffice?** It runs only for a new database.
3. **How handle jumping from v1 to v3?** Execute ordered steps for all missing versions.
4. **What is the key migration safety property?** Preserve valid user data or fail without partial upgrade.

# Try It Yourself

Create a v1 fixture with notes, add a `created_at` column in v2, and assert the old note remains after opening at v2.

# Challenge Project

Add a `category_id` relationship to an existing expense table. Migrate old rows into a default category and test the upgrade path.

# Quick Revision Sheet

- Version changes trigger migration logic.
- `onCreate` is for new installs; `onUpgrade` is for existing data.
- Apply sequential, transactional, tested transformations.
- Preserve rows and verify each supported upgrade path.

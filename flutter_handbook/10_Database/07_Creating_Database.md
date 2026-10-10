# Introduction

Creating a database means choosing a stable application-owned file path, opening that file, and establishing its initial schema. The file should live in the platform database directory, not a temporary or arbitrary working directory.

# Why this concept exists

The app must find the same database after relaunch. A stable path lets the SQLite engine reopen existing data, while versioned initialization makes first install and later upgrades predictable.

# Real-world analogy

Opening a database is like opening a named filing cabinet in a known office. If the cabinet is given a different name on every visit, the records appear to vanish because each visit opens a different cabinet.

# Internal Working

`getDatabasesPath()` returns the platform's database location. `openDatabase` opens or creates the file, then invokes the appropriate callback based on whether it is new or its schema version changed.

```text
resolve app database directory -> join stable filename -> openDatabase
                                                    -> onCreate (new file)
                                                    -> onUpgrade (version change)
```

# Syntax

```dart
final db = await openDatabase(
  path.join(await getDatabasesPath(), 'app.db'),
  version: 1,
  onCreate: (db, version) async => db.execute('CREATE TABLE ...'),
);
```

# Complete Example 1

The following app opens a named database and displays the resulting file path after creating its first table. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> createAppDatabase() async {
  final directory = await getDatabasesPath();
  final databaseFile = p.join(directory, 'database_creation_demo.db');
  return openDatabase(
    databaseFile,
    version: 1,
    onCreate: (db, version) async {
      await db.execute(
        'CREATE TABLE settings (id INTEGER PRIMARY KEY, label TEXT NOT NULL)',
      );
    },
  );
}

void main() => runApp(const MaterialApp(home: DatabasePage()));

class DatabasePage extends StatefulWidget {
  const DatabasePage({super.key});
  @override
  State<DatabasePage> createState() => _DatabasePageState();
}

class _DatabasePageState extends State<DatabasePage> {
  late final Future<Database> _database = createAppDatabase();
  String _status = 'Opening...';

  Future<void> _showStatus() async {
    try {
      final db = await _database;
      final count = Sqflite.firstIntValue(
            await db.rawQuery('SELECT COUNT(*) FROM settings'),
          ) ??
          0;
      if (mounted) setState(() => _status = 'Ready; $count settings');
    } catch (error) {
      if (mounted) setState(() => _status = 'Database error: $error');
    }
  }

  @override
  void initState() {
    super.initState();
    _showStatus();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Database setup')),
        body: Center(child: Text(_status)),
      );
}
```

**Line-by-line guide:** Imports supply Flutter, path joining, and SQLite. `createAppDatabase` asks the platform for its app database directory, joins a constant filename, and opens schema version 1. `onCreate` builds the initial `settings` table. The page caches the open future so rebuilds do not reopen it. `_showStatus` awaits it, runs a count query, converts the aggregate using `Sqflite.firstIntValue`, and shows a clear error if opening or querying fails. The `mounted` guard avoids updating a removed screen. `initState` starts initialization once. Widget tree: `MaterialApp -> DatabasePage -> Scaffold -> AppBar + Center/Text`.

**Database flow:** `stable path -> open/create file -> schema callback -> count query -> status state -> Text`.

**Execution:** The first launch creates the file/table; later launches reopen it and report readiness. If the filename changes, the app opens a new empty database. If the creation callback is removed, querying `settings` reports “no such table.”

```text
+----------------------+
| Database setup       |
|                      |
| Ready; 0 settings   |
+----------------------+
```

**Common mistakes:** Hard-coding a platform-specific absolute path, creating a random filename, or swallowing open errors.

**Professional use case:** A centralized opener provides one consistent handle and schema initialization point for all repositories.

# Complete Example 2

Use a reusable opener rather than repeating path construction:

```dart
class AppDatabase {
  AppDatabase(this.database);
  final Database database;

  static Future<AppDatabase> open() async {
    final file = p.join(await getDatabasesPath(), 'app_data.db');
    final database = await openDatabase(file, version: 1);
    return AppDatabase(database);
  }
}
```

The wrapper gives repositories a clear owner and single entry point. It does not yet create any tables, so calling it alone does not establish a useful schema.

# Complete Example 3

When a test needs a fresh database, use a test-specific temporary path and clean it up in test teardown. Never point test cleanup at a user's production database file. Unique test names prevent parallel tests from sharing state.

# Advanced Example

Configure database callbacks centrally:

```dart
openDatabase(
  file,
  version: 3,
  onCreate: createSchema,
  onUpgrade: migrateSchema,
  onConfigure: (db) async {
    await db.execute('PRAGMA foreign_keys = ON');
  },
);
```

Callbacks need to match the schema lifecycle. Apply connection-specific settings in `onConfigure`; write upgrades as safe, ordered changes.

# Under the Hood

The path identifies a file, not a schema. SQLite reads its metadata and schema version when opening. The package serializes operations for the handle according to its implementation and completes Dart futures with results or errors.

# Performance Notes

Open once per database service; opening is not a per-frame action. Keep initialization work bounded and move expensive data import into controlled, transactional setup.

# Best Practices

- Use `getDatabasesPath()` and a stable filename.
- Set an explicit integer schema version.
- Centralize callbacks and handle open failures.
- Keep initialization idempotent and migrations data-preserving.

# Common Mistakes

- Calling `openDatabase` inside `build`.
- Changing filename accidentally across app versions.
- Assuming `onCreate` executes for an existing file.
- Creating tables without a migration plan.

# Debugging

Display or log the resolved path in development. Inspect the file's `user_version`, table schema, and callback logs. Verify the app is not opening a second file under a slightly different name.

# Interview Questions

1. **Where should a mobile SQLite file live?** In the platform's application database directory.
2. **When does `onCreate` run?** When the database file is first created.
3. **Why use a version?** To select migration logic when the schema evolves.
4. **Why stable filenames?** To reopen the same persistent database.

# Try It Yourself

Add a second table and verify `onCreate` creates both tables on a clean install.

# Challenge Project

Build a database bootstrap class with foreign-key configuration, schema creation, migration hooks, and a test that opens a temporary database.

# Quick Revision Sheet

- Stable path + stable filename = persistent identity.
- `openDatabase` creates or opens the file.
- `onCreate` is first-install setup; `onUpgrade` is schema evolution.
- One centralized opener owns initialization.

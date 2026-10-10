# Introduction

A database is an organized place where an application stores information so it can find, change, and remove it later. A Flutter app might store notes, orders, products, or expense entries. This section focuses on databases—not preferences, authentication, or network APIs.

SQLite is a relational database that stores data in tables on the device. Flutter reaches it through a database package such as `sqflite`; Dart code issues operations and receives typed results asynchronously.

# Why this concept exists

Keeping records in ordinary variables is not enough: variables disappear when the process ends, and large collections become difficult to search or update safely. A database provides durable storage, structured rules, queries, indexes, and transactions.

# Real-world analogy

Think of a database as a well-run library. Tables are catalog sections, rows are books, columns describe each book, and an index helps the librarian find a title quickly without searching every shelf.

# Internal Working

The Flutter widget does not directly manipulate a file. A repository or data-access object sends a request through the database package; the native SQLite engine reads or writes pages in the database file and returns rows or an error.

```text
Widget -> State/Controller -> Repository -> sqflite -> SQLite file
   ^                                                    |
   +--------- decoded result / updated state <----------+
```

Database work is asynchronous from Dart. When data changes, Flutter rebuilds widgets only if application state is updated; SQLite itself does not rebuild the UI.

# Syntax

```dart
final rows = await db.query(
  'notes',
  where: 'id = ?',
  whereArgs: [noteId],
);
```

`?` is a bound parameter. Keep values separate from SQL text; this prevents quoting bugs and SQL injection.

# Complete Example 1

This runnable Flutter screen creates a database, inserts a note, then reads and displays all notes. Add `sqflite` and `path` to the Flutter app dependencies. SQLite platform setup is handled by the package for supported mobile targets.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as path;
import 'package:sqflite/sqflite.dart';

Future<Database> openNotesDatabase() async {
  final file = path.join(await getDatabasesPath(), 'intro_notes.db');
  return openDatabase(
    file,
    version: 1,
    onCreate: (db, version) async {
      await db.execute(
        'CREATE TABLE notes (id INTEGER PRIMARY KEY AUTOINCREMENT, title TEXT NOT NULL)',
      );
    },
  );
}

void main() => runApp(const MaterialApp(home: NotesPage()));

class NotesPage extends StatefulWidget {
  const NotesPage({super.key});

  @override
  State<NotesPage> createState() => _NotesPageState();
}

class _NotesPageState extends State<NotesPage> {
  late final Future<Database> _database = openNotesDatabase();
  List<Map<String, Object?>> _notes = [];

  Future<void> _load() async {
    final db = await _database;
    final rows = await db.query('notes', orderBy: 'id DESC');
    if (mounted) setState(() => _notes = rows);
  }

  Future<void> _add() async {
    final db = await _database;
    await db.insert('notes', {'title': 'A database-backed note'});
    await _load();
  }

  @override
  void initState() {
    super.initState();
    _load();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Database notes')),
        body: ListView(
          children: [
            for (final note in _notes) ListTile(title: Text(note['title']! as String)),
          ],
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: _add,
          child: const Icon(Icons.add),
        ),
      );
}
```

**Line-by-line guide:** Imports provide Material widgets, safe platform database paths, and SQLite APIs. `openNotesDatabase` resolves the application database directory and opens a named file at schema version 1. `onCreate` runs only when that file is first created; the table requires a title and generates its integer ID. `main` starts the screen. The state owns one database future and the currently displayed row maps. `_load` awaits the handle, queries rows newest-first, and guards `setState` with `mounted`. `_add` inserts through the package's map-based helper and reloads. `initState` triggers the first load once. The widget tree is `MaterialApp -> NotesPage -> Scaffold -> AppBar + ListView/ListTile + FAB`.

**Database flow:** `tap + -> insert map -> SQLite constraint checks -> SELECT -> state update -> ListView rebuild`.

**Execution:** On launch the database opens or is created, then the first query populates the list. Pressing `+` inserts one row; reloading displays it. The ASCII preview is:

```text
+----------------------+
| Database notes       |
|----------------------|
| A database-backed... |
|                      |
|                 [+]  |
+----------------------+
```

The database file makes the note survive app restarts. Without `onCreate`, the first query fails because the table does not exist; without `_load()` after insertion, the database changes but the visible list remains stale.

**Common mistakes:** Do not interpolate a title into SQL; use `insert` or placeholders. Do not open a new database on every build. Do not assume a database has been created just because the app screen rendered.

**Professional use case:** Offline notes, drafts, or cached reference data that must remain available after closing the app.

# Complete Example 2

A parameterized lookup can be executed with `db.query('notes', where: 'title = ?', whereArgs: [searchText])`. SQLite binds `searchText` as data rather than treating it as SQL. Inspect the returned list, handle an empty list as “not found,” and keep the query inside a repository so widgets do not know SQL details. Removing `whereArgs` while interpolating user text introduces injection and escaping defects.

# Complete Example 3

To count records, execute `SELECT COUNT(*) AS total FROM notes` with `rawQuery`, then read `rows.single['total']`. `COUNT(*)` returns one aggregate row, even when no notes match (the count is zero). A count query avoids loading every title into Dart just to call `.length`.

# Advanced Example

Use a transaction when several related writes must succeed or fail together:

```dart
await db.transaction((txn) async {
  await txn.insert('notes', {'title': 'Parent'});
  await txn.insert('audit_log', {'action': 'note-created'});
});
```

Both writes commit together. Use the transaction handle `txn` inside the callback; calling the outer `db` there can deadlock.

# Under the Hood

SQLite stores tables as B-tree structures in pages. A query is parsed, planned, and executed; a write updates pages and journal/WAL data according to database configuration. The package bridges Dart calls to the platform SQLite implementation.

# Performance Notes

Select only needed columns, filter in SQL, and add indexes for frequently searched columns after measuring. Avoid reading a whole large table into memory.

# Best Practices

- Put SQL behind a repository or database service.
- Use placeholders/bound arguments for values.
- Define constraints such as `NOT NULL`, `UNIQUE`, and foreign keys intentionally.
- Treat schema changes as versioned migrations.
- Keep database handles long-lived and close them when their owner is disposed.

# Common Mistakes

- Doing database I/O in `build`.
- Omitting primary keys or constraints.
- Assuming map values have the expected Dart type without validating them.
- Ignoring errors or presenting an empty result as success.

# Debugging

Log the operation and safe, non-sensitive identifiers; inspect the schema with a SQLite browser or `PRAGMA table_info(notes)`. Check that the actual database path and schema version are the expected ones.

# Interview Questions

1. **What is a table row?** One record whose values correspond to table columns.
2. **Why use a primary key?** It uniquely identifies a row and supports stable references.
3. **Why parameterize SQL?** Values stay data, avoiding injection and quoting errors.
4. **Does SQLite update Flutter UI automatically?** No; application state must be refreshed.

# Try It Yourself

Add a `created_at` integer column, display it, and sort newest-first. Verify the table survives a full app restart.

# Challenge Project

Build a local reading list with add, search, update, and delete actions. Keep SQL in a repository and show loading, empty, and error UI states.

# Quick Revision Sheet

- Database: durable, structured storage.
- SQLite: embedded relational database stored locally.
- Table -> rows -> columns.
- Bound parameters protect values.
- Database mutation and widget repaint are separate steps.
- Transactions group dependent writes atomically.

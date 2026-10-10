# Introduction

SQLite is a small relational database engine embedded in an application. It stores tables and indexes in a file and supports SQL, transactions, constraints, and joins. In Flutter, packages such as `sqflite` expose SQLite operations to Dart.

# Why this concept exists

Many mobile apps need dependable structured storage without running a database server on the device. SQLite provides durable, queryable storage with little operational setup.

# Real-world analogy

SQLite is like a filing cabinet that travels with the app. It has labeled drawers (tables), uniquely numbered folders (rows), and an index for quick retrieval.

# Internal Working

Flutter calls the Dart package, which bridges operations to a platform SQLite engine. SQLite reads or updates the database file and returns result rows asynchronously to Dart.

```text
Flutter -> sqflite method -> platform SQLite -> database file
        <- Future result <- rows / affected count / error
```

# Syntax

```sql
CREATE TABLE books (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT NOT NULL
);
```

`PRIMARY KEY` uniquely identifies a row; `NOT NULL` rejects missing titles.

# Complete Example 1

The app below inserts and lists books. Add `sqflite` and `path`; run on a platform supported by `sqflite`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openBookDb() async {
  final dbFile = p.join(await getDatabasesPath(), 'sqlite_intro.db');
  return openDatabase(dbFile, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE books (id INTEGER PRIMARY KEY AUTOINCREMENT, title TEXT NOT NULL)',
    );
  });
}

void main() => runApp(const MaterialApp(home: BooksPage()));

class BooksPage extends StatefulWidget {
  const BooksPage({super.key});
  @override
  State<BooksPage> createState() => _BooksPageState();
}

class _BooksPageState extends State<BooksPage> {
  late final Future<Database> _db = openBookDb();
  List<Map<String, Object?>> _books = [];

  Future<void> _load() async {
    final rows = await (await _db).query('books', orderBy: 'id DESC');
    if (mounted) setState(() => _books = rows);
  }

  Future<void> _add() async {
    await (await _db).insert('books', {'title': 'SQLite on Flutter'});
    await _load();
  }

  @override
  void initState() {
    super.initState();
    _load();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('SQLite bookshelf')),
        body: ListView(
          children: _books
              .map((book) => ListTile(title: Text(book['title']! as String)))
              .toList(),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: _add,
          child: const Icon(Icons.add),
        ),
      );
}
```

**Line-by-line guide:** The imports provide Flutter UI, path joining, and `sqflite`. `openBookDb` resolves the app database directory and opens a stable file. `version: 1` identifies the schema version; `onCreate` defines the initial table. The page retains the open database future and fetched rows. `_load` reads records newest-first and calls `setState` only if mounted. `_add` inserts a title then refreshes. `initState` triggers the first read. `build` renders the current rows and a button. Widget tree: `MaterialApp -> BooksPage -> Scaffold -> AppBar + ListView/ListTile + FAB`.

**Database flow:** `FAB -> sqflite insert -> SQLite constraints/file -> query -> row maps -> setState`.

**Execution:** First launch creates the file and table. Later launches open the same database. Removing the version/table setup results in “no such table”; removing refresh leaves the displayed list out of date.

```text
+----------------------+
| SQLite bookshelf     |
|----------------------|
| SQLite on Flutter    |
|                 [+]  |
+----------------------+
```

**Common mistakes:** Creating a new file path on each run, interpolating SQL, or forgetting that SQLite stores booleans as integer conventions.

**Professional use case:** Offline records such as transactions, saved drafts, and searchable catalogs.

# Complete Example 2

Read one row by primary key:

```dart
final rows = await db.query(
  'books',
  where: 'id = ?',
  whereArgs: [bookId],
  limit: 1,
);
```

SQLite uses the primary key to locate the row. `limit: 1` communicates that one result is expected. Check `rows.isEmpty` before accessing `single`; removing `where` can accidentally return a different book.

# Complete Example 3

Aggregate without loading every row:

```dart
final rows = await db.rawQuery('SELECT COUNT(*) AS total FROM books');
final total = rows.single['total'] as int;
```

The query returns one count value, reducing memory use. Do not cast to `int` blindly if the chosen driver or expression returns another numeric type; validate/conversion should follow the package's documented result types.

# Advanced Example

A foreign-key declaration can enforce related records:

```sql
PRAGMA foreign_keys = ON;
CREATE TABLE authors (
  id INTEGER PRIMARY KEY,
  name TEXT NOT NULL
);
CREATE TABLE books (
  id INTEGER PRIMARY KEY,
  author_id INTEGER NOT NULL REFERENCES authors(id),
  title TEXT NOT NULL
);
```

Enable foreign-key enforcement for each connection as required by the integration, and define delete behavior deliberately.

# Under the Hood

SQLite is serverless in the sense that the engine runs inside the application process; it is not a shared network database server. It implements SQL through a query planner and B-tree structures, with journaling or WAL behavior for crash consistency.

# Performance Notes

Use indexes for measured frequent lookups, projections to fetch only needed columns, and transactions for batches. Avoid huge result sets in widget state.

# Best Practices

- Store money as integer minor units, not binary floating-point.
- Use stable primary keys and explicit constraints.
- Plan migrations before changing a released schema.
- Use parameterized queries and transactions.

# Common Mistakes

- Assuming `onCreate` reruns after schema changes.
- Omitting a `where` clause on update/delete.
- Treating SQLite as a multi-device synchronization service.
- Ignoring `Future` errors.

# Debugging

Inspect database file path, schema version, `sqlite_master`, and table rows. Log SQL operation names and affected row counts, not private user content.

# Interview Questions

1. **What is SQLite?** An embedded relational database engine.
2. **What does `onCreate` do?** Initializes a newly created database at the first schema version.
3. **Why use a transaction?** To make a group of operations atomic.
4. **Is SQLite a remote server?** No; it is embedded and stores data locally.

# Try It Yourself

Add `author` to the book model and query books by author. Decide whether the author should be a separate table.

# Challenge Project

Build a searchable book library with add, detail, edit, delete, and a migration that adds a publication year.

# Quick Revision Sheet

- SQLite: embedded SQL engine and local file.
- `sqflite`: Flutter/Dart integration on supported platforms.
- `onCreate` is for first schema creation; `onUpgrade` handles later versions.
- Parameterize values; transact related writes.
- UI state refresh is application responsibility.

# Introduction

SQLite is an embedded relational database engine. It stores structured data in tables with rows and columns and supports SQL queries, indexes, constraints, and transactions.

# Why this concept exists

When data is relational, searchable, or too large for simple preferences, a database is a better fit. Examples include tasks, notes, inventory, and offline records.

# Real-world analogy

SQLite is a set of linked spreadsheets with rules. Tables hold records, IDs identify rows, and queries find or combine the records you need.

# Internal Working

SQLite runs in-process and stores database pages in a file. A Flutter plugin exposes operations through Dart APIs. SQL statements are parameterized to separate query structure from user data.

Transactions group related operations. Migrations update schema versions across app releases. Reads and writes may use asynchronous Dart APIs; keep expensive result processing off the UI thread where necessary.

# Syntax

```sql
CREATE TABLE notes (
  id INTEGER PRIMARY KEY,
  title TEXT NOT NULL,
  body TEXT NOT NULL
);
```

# Complete Example 1

This complete UI demonstrates local persistence through a repository interface, while the concrete SQLite driver is intentionally left as a replaceable boundary. For a real implementation, use a maintained plugin such as `sqflite`, add it to dependencies, and implement the repository with parameterized SQL. This avoids presenting plugin-version-specific code as universal.

```dart
import 'package:flutter/material.dart';

class Note {
  const Note({required this.id, required this.title});
  final int id;
  final String title;
}

abstract interface class NoteStore {
  Future<List<Note>> loadNotes();
  Future<void> addNote(String title);
}

class DemoNoteStore implements NoteStore {
  final List<Note> _notes = [];
  int _nextId = 1;

  @override
  Future<List<Note>> loadNotes() async => List.unmodifiable(_notes);

  @override
  Future<void> addNote(String title) async {
    _notes.add(Note(id: _nextId++, title: title));
  }
}

void main() => runApp(const SqliteIntroApp());

class SqliteIntroApp extends StatelessWidget {
  const SqliteIntroApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: SqliteIntroPage());
}

class SqliteIntroPage extends StatefulWidget {
  const SqliteIntroPage({super.key});

  @override
  State<SqliteIntroPage> createState() => _SqliteIntroPageState();
}

class _SqliteIntroPageState extends State<SqliteIntroPage> {
  final NoteStore _store = DemoNoteStore();
  final _input = TextEditingController();
  List<Note> _notes = [];

  Future<void> _reload() async {
    final notes = await _store.loadNotes();
    if (mounted) setState(() => _notes = notes);
  }

  Future<void> _add() async {
    final title = _input.text.trim();
    if (title.isEmpty) return;
    await _store.addNote(title);
    _input.clear();
    await _reload();
  }

  @override
  void dispose() {
    _input.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('SQLite concepts')),
        body: Column(
          children: [
            TextField(
              controller: _input,
              decoration: const InputDecoration(labelText: 'Note title'),
            ),
            FilledButton(onPressed: _add, child: const Text('Add')),
            Expanded(
              child: ListView(
                children: _notes.map((note) => ListTile(title: Text(note.title))).toList(),
              ),
            ),
          ],
        ),
      );
}
```

### Line-by-line explanation

- `Note` is the domain model exposed to the UI.
- `NoteStore` defines persistence operations without tying the widget to one database plugin.
- `DemoNoteStore` is in-memory only and deliberately demonstrates the boundary, not durable SQLite.
- `_reload` loads a snapshot and updates state if mounted.
- `_add` validates input, calls the store, clears the field, and refreshes the list.
- The UI provides a form and list; `ListView` displays records.
- `dispose` releases the text controller.

### Widget tree

```text
SqliteIntroPage → Scaffold → Column
                           ├→ TextField
                           ├→ Add button
                           └→ Expanded → ListView → ListTile*
```

### Execution flow

```text
Widget → NoteStore interface → SQLite repository in production
Add → INSERT transaction → query notes → render result
```

### ASCII phone preview

```text
+---------------------------+
|     SQLite concepts       |
+---------------------------+
| Note title                |
| [ Call dentist         ]  |
| [          Add         ]  |
| • Call dentist            |
+---------------------------+
```

### Example explanation

The code is explicitly runnable but its demo store is not persistent. A real SQLite adapter implements `NoteStore` and maps `Note` to a table. This separation lets UI behavior be exercised without pretending the in-memory fake writes to disk.

If `DemoNoteStore` is used in production, data disappears on restart. A concrete implementation must parameterize user values and manage database lifecycle/migrations.

**Common mistakes:** Do not concatenate user text into SQL. Do not run schema creation on every screen build.

**Professional use case:** Searchable offline records with relationships and transactions.

# Complete Example 2

Parameterized insertion:

```sql
INSERT INTO notes (title, body) VALUES (?, ?);
```

The database driver binds values to placeholders, preventing user content from becoming SQL syntax.

# Complete Example 3

Query only needed records:

```sql
SELECT id, title FROM notes
WHERE title LIKE ?
ORDER BY title
LIMIT ?;
```

Indexes and query plans should be considered as data grows.

# Advanced Example

Use schema versioning and migrations, enable foreign-key constraints where needed, and wrap multi-step writes in transactions. Test upgrade paths from every supported prior schema.

# Under the Hood

SQLite uses pages, a journaling mode, a query planner, and B-tree indexes. A transaction defines an atomic group of operations. A Flutter plugin adapts these native database capabilities to Dart.

# Performance Notes

- Select only necessary columns.
- Add indexes for frequent predicates and ordering.
- Avoid N+1 query loops.
- Paginate large results.
- Use transactions for batches.

# Best Practices

- Use parameterized SQL.
- Give tables primary keys and constraints.
- Centralize database access in repositories/DAOs.
- Version and migrate schema.
- Handle errors and test real upgrade paths.

# Common Mistakes

- SQL string concatenation.
- Missing `WHERE` predicates.
- No migration plan.
- Doing full-table reads by default.
- Treating database entities as widget state without a repository.

# Debugging

- Inspect schema and database version.
- Log SQL structure and timings, not private data.
- Verify migrations from older versions.
- Test constraints and transaction rollback.
- Use explain/query-plan tools for slow queries.

# Interview Questions

1. Why choose SQLite over preferences? Relational data, queries, constraints, and transactions.
2. What is a parameterized query? SQL with bound values separate from query syntax.
3. What is a migration? A versioned schema transformation.
4. Why use an index? Speeding up selected lookups at storage/write cost.
5. What does a transaction guarantee? Atomic grouping of operations under database semantics.

# Try It Yourself

1. Design a notes table with a primary key and timestamp.
2. Write a parameterized insert and filtered select.
3. Add an index and identify the query it benefits.
4. Plan a migration that adds a nullable field.

# Challenge Project

Implement a notes repository using SQLite with create/read/update/delete, search, migration tests, and parameterized queries. Keep SQL and UI separate.

# Quick Revision Sheet

- SQLite is embedded relational storage.
- Tables/rows/columns, indexes, constraints, transactions.
- Bind values; never concatenate untrusted input.
- Migrate schema across releases.
- Query only what is needed and measure plans.

# Introduction

The repository pattern places a stable data-access interface between application logic and a database implementation. A screen asks for tasks; the repository decides how those tasks are loaded or saved.

# Why this concept exists

When widgets contain SQL, database details spread throughout the UI and make replacement, testing, and error handling difficult. A repository centralizes persistence behavior.

# Real-world analogy

A repository is a service desk. The customer asks for a record by name; the clerk knows which archive shelf and filing procedure to use.

# Internal Working

The UI calls an abstract contract. A concrete SQLite repository maps domain operations to `sqflite`. Results return as typed objects, and UI state changes after awaited operations complete.

```text
Widget -> TaskRepository interface -> SqfliteTaskRepository -> SQLite
       <- typed Task/results        <- query maps             <- file
```

# Syntax

```dart
abstract interface class TaskRepository {
  Future<List<Task>> getAll();
  Future<int> add(String title);
}
```

# Complete Example 1

The app uses a repository for a simple SQLite-backed task list. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

class Task {
  const Task({required this.id, required this.title});
  final int id;
  final String title;
}

abstract interface class TaskRepository {
  Future<List<Task>> getAll();
  Future<void> add(String title);
}

class SqfliteTaskRepository implements TaskRepository {
  SqfliteTaskRepository(this._db);
  final Database _db;

  @override
  Future<List<Task>> getAll() async {
    final rows = await _db.query('tasks', orderBy: 'id DESC');
    return rows
        .map((row) => Task(
              id: row['id']! as int,
              title: row['title']! as String,
            ))
        .toList();
  }

  @override
  Future<void> add(String title) async {
    await _db.insert('tasks', {'title': title});
  }
}

Future<Database> openDb() async {
  final file = p.join(await getDatabasesPath(), 'repository_demo.db');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE tasks (id INTEGER PRIMARY KEY AUTOINCREMENT, title TEXT NOT NULL)',
    );
  });
}

void main() => runApp(const MaterialApp(home: RepositoryPage()));

class RepositoryPage extends StatefulWidget {
  const RepositoryPage({super.key});
  @override
  State<RepositoryPage> createState() => _RepositoryPageState();
}

class _RepositoryPageState extends State<RepositoryPage> {
  late final Future<TaskRepository> _repository =
      openDb().then(SqfliteTaskRepository.new);
  List<Task> _tasks = [];

  Future<void> _load() async {
    final repository = await _repository;
    final tasks = await repository.getAll();
    if (mounted) setState(() => _tasks = tasks);
  }

  Future<void> _add() async {
    final repository = await _repository;
    await repository.add('Saved through repository');
    await _load();
  }

  @override
  void initState() {
    super.initState();
    _load();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Repository pattern')),
        body: ListView(
          children: _tasks.map((task) => ListTile(title: Text(task.title))).toList(),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: _add,
          child: const Icon(Icons.add),
        ),
      );
}
```

**Line-by-line guide:** `Task` is the UI/domain representation. `TaskRepository` is the contract. `SqfliteTaskRepository` implements it by querying rows and mapping each row to a typed task; `add` hides insert details. `openDb` initializes the local schema. The page caches a future repository, loads typed tasks, adds through the contract, and refreshes. `mounted` prevents stale async updates. Widget tree: `MaterialApp -> RepositoryPage -> Scaffold -> ListView/ListTile + FAB`.

**Database flow:** `UI intent -> repository method -> SQLite statement -> typed Task -> state -> widgets`.

**Execution:** The widget never knows table names. A test can substitute another implementation of the contract. Removing the repository moves SQL into the page and couples UI to `sqflite`; removing mapping exposes dynamic row maps to widgets.

```text
+----------------------+
| Repository pattern   |
|----------------------|
| Saved through...     |
|                 [+]  |
+----------------------+
```

**Common mistakes:** Creating an abstraction with no meaningful boundary, returning dynamic maps, and making repository methods silently ignore failures.

**Professional use case:** Keep screens stable while persistence moves from an in-memory test double to a local SQL database.

# Complete Example 2

A test fake implements the same interface:

```dart
class InMemoryTaskRepository implements TaskRepository {
  final List<Task> _tasks = [];
  var _nextId = 1;

  @override
  Future<List<Task>> getAll() async => List.unmodifiable(_tasks);

  @override
  Future<void> add(String title) async {
    _tasks.add(Task(id: _nextId++, title: title));
  }
}
```

This fake supports fast unit tests, not durable storage. It lets application behavior be tested without opening SQLite.

# Complete Example 3

A repository should expose domain actions such as `findById`, `search`, or `delete`, rather than a generic raw SQL method used by every widget. Explain each method's empty/missing/error behavior in its contract.

# Advanced Example

Inject the repository through the app's composition root or constructor. Keep database opening, migration, and close ownership outside individual screens. Do not create multiple repositories that each open the same file without a lifecycle plan.

# Under the Hood

The repository is a software boundary, not a database feature. It maps database-specific rows/errors into application types and can centralize transactions, mapping, and query policies.

# Performance Notes

Avoid per-row repository calls for lists; expose an efficient query that returns the needed page or aggregate. Do not over-generalize away useful database features.

# Best Practices

- Define interfaces around app use cases.
- Map rows into typed models at the boundary.
- Make failures and affected-row outcomes explicit.
- Use fakes for unit tests and real SQLite for integration tests.

# Common Mistakes

- Putting SQL back into widgets through a “generic” repository method.
- Returning raw `Map` values as the public contract.
- Swallowing exceptions or returning fake success.
- Adding abstractions that merely rename one method without improving testability or ownership.

# Debugging

Test repository methods against a temporary database and inspect exact SQL arguments. Trace one use case from widget action to returned domain result.

# Interview Questions

1. **What is the repository pattern?** A boundary that centralizes access to data sources.
2. **Why use it with a database?** It decouples UI/application logic from SQL and enables focused tests.
3. **Should repositories return raw maps?** Usually no; typed domain/data models make contracts clearer.
4. **How test a repository?** Unit test mapping/contracts and integration test it against a temporary real database.

# Try It Yourself

Add update and delete methods to the interface and both implementations.

# Challenge Project

Refactor a screen with embedded SQL into a repository, then test the same UI behavior using an in-memory fake.

# Quick Revision Sheet

- Widget depends on a repository contract.
- Concrete adapter owns database mapping.
- Return typed records and meaningful outcomes.
- Separate unit fakes from database integration tests.

# Introduction

Isar is a local database designed for fast object-oriented storage in Flutter/Dart. It offers typed collections and indexes; exact APIs and platform support depend on the Isar package generation/version selected by a project.

# Why this concept exists

Some apps work primarily with object collections and want indexed local queries without writing SQL for every operation. An object database maps Dart model objects into stored records.

# Real-world analogy

An object database is like a warehouse where each item is stored as a labeled object with selected lookup tags, rather than as rows in hand-designed relational tables.

# Internal Working

The app defines a model and generated schema metadata. A database instance opens collections, stores objects, and executes indexed queries. Writes should be grouped in write transactions according to the package API.

```text
Dart model -> generated collection schema -> object transaction -> local store
           <- typed object query <--------- index lookup
```

# Syntax

```dart
// Conceptual Isar pattern; constructor and generator APIs vary by release.
await isar.writeTxn(() async {
  await isar.tasks.put(task);
});
```

# Complete Example 1

The following is an Isar 3-style illustrative Flutter example. Add compatible `isar`, `isar_flutter_libs`, `isar_generator`, `build_runner`, and `path_provider` dependencies. Pin versions together and follow that release's generator/platform setup; Isar releases have changed setup APIs. Generate the collection schema before compiling.

```dart
import 'package:flutter/material.dart';
import 'package:isar/isar.dart';
import 'package:path_provider/path_provider.dart';

part 'main.g.dart';

@collection
class Task {
  Id id = Isar.autoIncrement;
  late String title;
}

Future<Isar> openTaskStore() async {
  final directory = await getApplicationDocumentsDirectory();
  return Isar.open([TaskSchema], directory: directory.path);
}

void main() => runApp(const MaterialApp(home: IsarPage()));

class IsarPage extends StatefulWidget {
  const IsarPage({super.key});
  @override
  State<IsarPage> createState() => _IsarPageState();
}

class _IsarPageState extends State<IsarPage> {
  late final Future<Isar> _isar = openTaskStore();
  List<Task> _tasks = [];

  Future<void> _refresh() async {
    final isar = await _isar;
    final tasks = await isar.tasks.where().findAll();
    if (mounted) setState(() => _tasks = tasks);
  }

  Future<void> _add() async {
    final isar = await _isar;
    final task = Task()..title = 'Object database record';
    await isar.writeTxn(() async => isar.tasks.put(task));
    await _refresh();
  }

  @override
  void initState() {
    super.initState();
    _refresh();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Isar tasks')),
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

**Line-by-line guide:** Imports supply Flutter, Isar annotations/APIs, and an application-owned directory. `@collection` marks a model for schema generation; `Id` is the record identifier. `openTaskStore` resolves a durable directory and opens the generated schema. The page retains the asynchronous database handle and typed records. `_refresh` reads all tasks and updates state while mounted. `_add` constructs an object, writes it in a write transaction, then refreshes. `initState` loads initial data; `build` renders tasks and an add button. Widget tree: `MaterialApp -> IsarPage -> Scaffold -> ListView/ListTile + FAB`.

**Database flow:** `Task object -> write transaction -> Isar collection -> typed query -> state -> list`.

**Execution:** The generated `TaskSchema` is required at build time. A tap persists a task, then the query updates the list. Removing the write transaction violates the package's required write pattern; removing generated schema setup makes the code incomplete.

```text
+----------------------+
| Isar tasks           |
|----------------------|
| Object database...   |
|                 [+]  |
+----------------------+
```

**Common mistakes:** Copying setup instructions from a different Isar major version, skipping code generation, and loading an unbounded collection into UI state.

**Professional use case:** High-volume local object records with indexed lookups, when object-oriented access fits the application's data model.

# Complete Example 2

An indexed query can retrieve records by a frequently searched property after defining the appropriate index in the model and regenerating code. The exact annotation and generated query syntax vary between Isar releases; consult the pinned release documentation and verify that the index is actually present in the generated schema.

# Complete Example 3

Update and delete operations should occur in a write transaction. Query by stable ID, handle a missing object, mutate the model, and persist it. Never assume assigning a Dart field alone updates storage.

# Advanced Example

Model links carefully. Object links can be convenient, but they do not remove the need to understand ownership, deletion, transaction scope, indexes, and migration behavior. Benchmark query patterns using production-like collection sizes.

# Under the Hood

The generator produces schema metadata that lets Isar encode typed objects and execute queries. The database maintains its own storage structures and indexes; a Dart object instance is not automatically a live stored object.

# Performance Notes

Use indexes for real query paths, limit results, and avoid expensive object deserialization when only a count or aggregate is needed.

# Best Practices

- Pin a compatible Isar version and generator.
- Keep model schema changes explicit and tested.
- Group mutations in write transactions.
- Verify platform support and maintenance status before selecting a database for a new production app.

# Common Mistakes

- Mixing code snippets from incompatible major versions.
- Forgetting generation/part files.
- Assuming model mutation persists automatically.
- Using an index without a query need.

# Debugging

Check package version, generator output, opened directory, schema registration, and transaction errors. Reproduce with a minimal model before investigating complex links.

# Interview Questions

1. **What is Isar?** A local object-oriented database option for Flutter/Dart.
2. **Why generate schema code?** The engine needs metadata describing persisted models.
3. **Why write transactions?** They define safe boundaries for mutations.
4. **What should be checked before adopting it?** Version compatibility, platform support, maintenance, migration, and workload fit.

# Try It Yourself

Add a boolean completion field and an index for a common filter using the exact pinned version's API.

# Challenge Project

Prototype an offline task collection in Isar, then compare its query ergonomics and migration story with a SQLite implementation.

# Quick Revision Sheet

- Isar stores typed object collections locally.
- Generated schema and version-specific setup matter.
- Mutations use write transactions.
- Confirm current package support and migration behavior before production adoption.

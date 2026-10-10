# Introduction

ObjectBox is an object-oriented local database with a Flutter/Dart integration. It stores typed entities and provides generated accessors, queries, and relations. Consult the selected package version for setup and platform details.

# Why this concept exists

Object-centric apps may benefit from persisting model objects directly instead of manually mapping every field to SQL maps. ObjectBox provides entity metadata and database operations around that model.

# Real-world analogy

ObjectBox is a labeled storage system where typed objects go into a collection and generated lookup tools retrieve matching objects.

# Internal Working

Annotations describe entities; code generation creates model metadata and accessors. A `Store` owns the database, and boxes provide typed operations on entity records. The exact API should match the package version in use.

```text
Entity declaration -> generated model -> Store -> Box.put/query -> local database
```

# Syntax

```dart
// Illustrative ObjectBox pattern; generated identifiers depend on setup.
final id = box.put(entity);
final entity = box.get(id);
```

# Complete Example 1

This example demonstrates the common ObjectBox entity/box pattern. ObjectBox setup and generated `objectbox.g.dart` APIs can differ by package version; add the compatible Flutter package and run its generator before compiling.

```dart
import 'package:flutter/material.dart';
import 'package:objectbox/objectbox.dart';
import 'objectbox.g.dart';

@Entity()
class LocalTask {
  @Id()
  int id = 0;
  String title;

  LocalTask(this.title);
}

late final Store store;

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  store = await openStore();
  runApp(const MaterialApp(home: ObjectBoxPage()));
}

class ObjectBoxPage extends StatefulWidget {
  const ObjectBoxPage({super.key});
  @override
  State<ObjectBoxPage> createState() => _ObjectBoxPageState();
}

class _ObjectBoxPageState extends State<ObjectBoxPage> {
  late final Box<LocalTask> _tasks = store.box<LocalTask>();
  List<LocalTask> _items = [];

  void _refresh() => setState(() => _items = _tasks.getAll());

  void _add() {
    _tasks.put(LocalTask('ObjectBox record'));
    _refresh();
  }

  @override
  void initState() {
    super.initState();
    _items = _tasks.getAll();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('ObjectBox tasks')),
        body: ListView(
          children: _items.map((item) => ListTile(title: Text(item.title))).toList(),
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: _add,
          child: const Icon(Icons.add),
        ),
      );
}
```

**Line-by-line guide:** Imports provide widgets, ObjectBox annotations/types, and generated store access. `@Entity` marks a persisted class; `@Id` identifies its generated key, where zero commonly requests a new ID. `main` initializes Flutter bindings, opens the generated store, and then launches the app. The page gets a typed box and loads entities. `_add` persists an object with `put`, then refreshes state. `build` maps entities to list tiles. Widget tree: `MaterialApp -> ObjectBoxPage -> Scaffold -> AppBar + ListView/ListTile + FAB`.

**Database flow:** `LocalTask -> Box.put -> ObjectBox store -> getAll -> state -> ListView`.

**Execution:** Generation must produce the store opener and entity metadata. Tapping the button persists a task. Removing `put` only creates a Dart object and loses it when the method ends; removing store initialization leaves the box unavailable.

```text
+----------------------+
| ObjectBox tasks      |
|----------------------|
| ObjectBox record     |
|                 [+]  |
+----------------------+
```

**Common mistakes:** Forgetting generated model code or opening the store before using boxes.

**Professional use case:** Persisting local entity models with generated object access and relations.

# Complete Example 2

Update by loading an entity using its ID, changing its fields, and saving it again with `put`. In object databases, mutating a Dart object in memory is not equivalent to committing the mutation. Handle a missing ID explicitly.

# Complete Example 3

Delete a specific object by its database ID and refresh any displayed collection. Avoid clearing a whole box when the user intended to remove one record.

# Advanced Example

Define a relation when entities have associations, and inspect generated model metadata after schema changes. Use transactions or the package's bulk APIs for multi-object invariants, according to the pinned version's semantics.

# Under the Hood

Generated metadata describes persisted fields and identifiers. The store manages database access, while typed boxes expose entity-oriented CRUD and query operations.

# Performance Notes

Use indexes for frequent predicates, avoid repeated `getAll()` on large collections, and query only the required subset.

# Best Practices

- Pin package and generator versions together.
- Keep a single well-owned store lifecycle.
- Test schema updates and entity relations.
- Verify platform support and operational maturity for production use.

# Common Mistakes

- Editing generated files directly.
- Assuming a field mutation is automatically saved.
- Reopening stores repeatedly.
- Loading every entity for a small UI subset.

# Debugging

Check generated code, model annotations, store path, and whether writes complete before querying. Confirm the exact package version's transaction/query APIs.

# Interview Questions

1. **What is ObjectBox?** An object-oriented local database with Flutter/Dart bindings.
2. **What does a generated model do?** Describes entities to the database runtime.
3. **Why is `put` needed after edits?** To persist the changed entity.
4. **What should be verified before adopting it?** Version support, migration path, platform compatibility, and workload.

# Try It Yourself

Add a `done` field and query incomplete tasks using the selected version's query API.

# Challenge Project

Build a local inventory prototype with entities, indexes, and relations; document how upgrades preserve stored data.

# Quick Revision Sheet

- ObjectBox persists typed entities in a local store.
- Generated metadata is required.
- Save modifications through the database API.
- Match code to the exact package/generator version.

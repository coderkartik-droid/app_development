# Introduction

Drift is a reactive, type-safe persistence library for Dart and Flutter, commonly used with SQLite. It lets developers define tables and queries in Dart, generates database code, and can expose query results as streams.

# Why this concept exists

Raw SQL and dynamic row maps are flexible but can move schema mistakes to runtime. Drift adds typed table definitions, generated data classes, and query composition while retaining SQLite capabilities.

# Real-world analogy

Drift is like a database-aware compiler assistant: you describe the schema and operations, then generated code gives Dart names and types for interacting with them.

# Internal Working

Developers declare tables and a database class. A code generator produces row models and query plumbing. Runtime operations still use SQLite; a watched query emits updated results when relevant tables change.

```text
Dart schema + SQL query -> code generation -> typed DAO/database
                                      -> SQLite -> query stream -> UI
```

# Syntax

```dart
@DriftDatabase(tables: [Tasks])
class AppDatabase extends _$AppDatabase {
  AppDatabase(QueryExecutor e) : super(e);
  @override
  int get schemaVersion => 1;
}
```

# Complete Example 1

This example is a complete minimal Drift database definition; the Flutter page demonstrates a watched query. Add the current compatible `drift`, `drift_flutter` (or chosen executor), `drift_dev`, and `build_runner` dependencies, then run code generation. Generated files are produced by the normal build step and must not be hand-written.

```dart
import 'package:drift/drift.dart';
import 'package:drift_flutter/drift_flutter.dart';
import 'package:flutter/material.dart';

part 'main.g.dart';

class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
}

@DriftDatabase(tables: [Tasks])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(driftDatabase(name: 'drift_intro'));
  @override
  int get schemaVersion => 1;
}

void main() => runApp(const MaterialApp(home: DriftPage()));

class DriftPage extends StatefulWidget {
  const DriftPage({super.key});
  @override
  State<DriftPage> createState() => _DriftPageState();
}

class _DriftPageState extends State<DriftPage> {
  late final AppDatabase _database = AppDatabase();

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Drift tasks')),
        body: StreamBuilder<List<Task>>(
          stream: _database.select(_database.tasks).watch(),
          builder: (context, snapshot) {
            if (!snapshot.hasData) {
              return const Center(child: CircularProgressIndicator());
            }
            return ListView(
              children: snapshot.data!
                  .map((task) => ListTile(title: Text(task.title)))
                  .toList(),
            );
          },
        ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => _database.into(_database.tasks).insert(
                TasksCompanion.insert(title: 'Typed Drift row'),
              ),
          child: const Icon(Icons.add),
        ),
      );
}
```

**Line-by-line guide:** `drift` provides schema/query APIs, `drift_flutter` the selected Flutter database executor, and Flutter the UI. The `part` declaration connects generated code. `Tasks` defines an auto-increment integer ID and required title. `@DriftDatabase` asks the generator to create typed accessors. `AppDatabase` opens a named SQLite database and declares schema version 1. `StreamBuilder` listens to a watched query; it shows loading before the first result and list tiles after. The FAB inserts using a generated companion type. Widget tree: `MaterialApp -> DriftPage -> Scaffold -> StreamBuilder -> ListView/ListTile + FAB`.

**Database flow:** `insert typed companion -> SQLite write -> watched table invalidates query -> stream emits -> StreamBuilder rebuilds`.

**Execution:** Generate code before building. Pressing the button inserts a typed row; the watched query emits the new list. Removing `part` or generated output causes unresolved generated symbols; removing `.watch()` means the stream-driven refresh is lost.

```text
+----------------------+
| Drift tasks          |
|----------------------|
| Typed Drift row      |
|                 [+]  |
+----------------------+
```

**Common mistakes:** Forgetting code generation, mismatching the part filename, and failing to close long-lived database instances at their owner boundary.

**Professional use case:** A large local relational model with typed queries and screens that react to database changes.

# Complete Example 2

Insert a row from repository code with a generated companion:

```dart
await into(tasks).insert(TasksCompanion.insert(title: 'Read'));
```

Generated companions distinguish required insert values from optional/defaulted columns. The returned result and generated row type depend on the operation; use the generated API rather than dynamic maps where possible.

# Complete Example 3

Filter typed rows:

```dart
final query = select(tasks)..where((task) => task.title.like('%read%'));
final matches = await query.get();
```

The query builder composes a predicate and executes asynchronously. Escape wildcard characters or define search semantics if the search text is user-controlled.

# Advanced Example

Move complex queries into a DAO and test it with an in-memory executor where supported. Keep schema version/migration definitions aligned with generated output and verify migration behavior using an existing database fixture.

# Under the Hood

Code generation translates declarations into Dart APIs; it does not replace SQLite. Watched queries use table dependency tracking to decide when to rerun and emit new results.

# Performance Notes

Watch bounded queries and avoid expensive full-table stream invalidation for unrelated screens. Add indexes through schema declarations/migrations when a measured query needs them.

# Best Practices

- Run and verify code generation after schema edits.
- Group query logic in DAOs/repositories.
- Keep schema migrations under test.
- Own and close the database at an application/service boundary.

# Common Mistakes

- Treating generated source as hand-edited code.
- Changing schema without incrementing version.
- Watching a huge unfiltered table unnecessarily.
- Ignoring stream errors and loading states.

# Debugging

Check generator output, part paths, build_runner logs, schema version, and the generated SQL. Test watched query updates after each mutation.

# Interview Questions

1. **What does Drift add over raw `sqflite`?** Typed schema/query APIs, generated models, and reactive query streams.
2. **Does Drift stop using SQLite?** No; it uses SQLite through an executor.
3. **Why run code generation?** To create typed database and query code from declarations.
4. **What does `watch()` do?** Emits query results and reruns when relevant data changes.

# Try It Yourself

Add a `done` column and a DAO query for incomplete tasks; migrate a version 1 fixture.

# Challenge Project

Create a Drift notes repository with typed search, a watched list, a detail query, and migration tests.

# Quick Revision Sheet

- Drift = typed/reactive SQLite abstraction.
- Tables and queries generate Dart code.
- Watched queries emit after relevant database changes.
- Still plan SQLite schema, indexes, transactions, and migrations.

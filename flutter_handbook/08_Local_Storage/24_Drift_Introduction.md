# Introduction

Drift is a reactive, type-safe persistence library built around SQLite for Dart and Flutter. It lets developers define tables and queries in Dart and can generate strongly typed database APIs.

# Why this concept exists

Raw SQL is powerful but can become difficult to maintain when table names, columns, and mappings are repeated as strings. Drift adds generated types, migrations, and streams while retaining SQLite's relational engine.

# Real-world analogy

SQLite is the engine and storage room; Drift is the labeled control panel that checks which switches and connections are valid before operating the engine.

# Internal Working

Drift table declarations describe schema. Code generation produces data classes and database accessors. Queries can return futures or streams; streams notify subscribers when relevant tables change. Schema versions and migration strategies manage upgrades.

# Syntax

Illustrative Drift table declaration:

```dart
class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  BoolColumn get done => boolean().withDefault(const Constant(false))();
}
```

Drift syntax and platform setup evolve; add compatible `drift`, `drift_flutter`/platform support, and `drift_dev` dependencies as documented for the selected release.

# Complete Example 1

This code shows a typed Drift table and database declaration. It is a library example, not a standalone `main.dart`; add the listed dependencies, configure the platform database connection, and run code generation to create `app_database.g.dart`.

```dart
import 'package:drift/drift.dart';
part 'app_database.g.dart';

class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text()();
  BoolColumn get done => boolean().withDefault(const Constant(false))();
}

@DriftDatabase(tables: [Tasks])
class AppDatabase extends _$AppDatabase {
  AppDatabase(super.e);

  @override
  int get schemaVersion => 1;

  Future<int> addTask(String title) {
    return into(tasks).insert(TasksCompanion.insert(title: title));
  }

  Stream<List<Task>> watchTasks() {
    return (select(tasks)..orderBy([(task) => OrderingTerm.desc(task.id)])).watch();
  }
}
```

### Line-by-line explanation

- `drift.dart` supplies table, query, and database APIs.
- The `part` directive connects generated database code.
- `Tasks` declares an auto-increment integer primary key, required title, and default-false boolean.
- `@DriftDatabase` lists tables that participate in generated database access.
- `AppDatabase` extends generated `_$AppDatabase`.
- `schemaVersion` identifies the persisted schema version and must change with migrations.
- `addTask` inserts a typed companion rather than building SQL strings.
- `watchTasks` returns a stream that emits rows ordered by descending ID.

### Widget tree

```text
UI (StreamBuilder) → AppDatabase.watchTasks() → generated query → SQLite
```

### Execution flow

```text
Table declaration → code generator → typed database class
Insert task → generated query → SQLite
Database changes → watchTasks stream emits → UI rebuilds
```

### ASCII phone preview

```text
+---------------------------+
|        Drift tasks        |
+---------------------------+
| □ Read chapter             |
| ✓ Review notes             |
+---------------------------+
```

### Example explanation

The generated companion enforces required insert fields at compile time. The stream enables reactive UI without manually reloading after each write. This file still needs a database executor and generated output before the project compiles.

If `schemaVersion` is not advanced when schema changes, migration logic may not run. If code generation is skipped, `_$AppDatabase` and `TasksCompanion` are missing.

**Common mistakes:** Do not edit generated files; modify table/query declarations and rerun the generator.

**Professional use case:** Typed offline business data with reactive screens and evolving schema.

# Complete Example 2

Observe a query in a widget:

```dart
StreamBuilder<List<Task>>(
  stream: database.watchTasks(),
  builder: (context, snapshot) {
    if (!snapshot.hasData) return const CircularProgressIndicator();
    return TaskList(tasks: snapshot.data!);
  },
)
```

Handle `snapshot.hasError` in production and avoid recreating expensive streams on every build.

# Complete Example 3

Update a generated row:

```dart
await (update(tasks)..where((task) => task.id.equals(id)))
    .write(TasksCompanion(done: Value(true)));
```

The predicate scopes the update; `Value` indicates that the column should be changed.

# Advanced Example

Define migrations explicitly. Use a migration strategy that handles fresh creation, upgrades, and any required data transformation. Test each supported old schema version using fixtures; generated type safety does not guarantee data compatibility.

# Under the Hood

Drift code generation produces database classes, row models, companions, and query bindings. SQLite executes the actual SQL. Drift tracks table dependencies so streams can emit updates after relevant writes.

# Performance Notes

- Use query streams for reactive screens but avoid too many broad watchers.
- Select only needed columns.
- Add indexes for common filters/sorts.
- Use transactions for batches.
- Profile generated queries against realistic data volume.

# Best Practices

- Keep schema and queries in database modules.
- Rerun code generation after declaration changes.
- Increment schema version and implement tested migrations.
- Handle stream errors and loading.
- Keep UI independent of raw SQL.

# Common Mistakes

- Missing `part` or generated code.
- Failing to update schema version.
- Ignoring migration tests.
- Recreating a stream every build.
- Watching entire tables when a narrower query works.

# Debugging

- Run the configured build runner and inspect generation errors.
- Verify executor/platform setup.
- Inspect schema version and migration path.
- Handle stream errors explicitly.
- Compare generated query output with expected rows.

# Interview Questions

1. What does Drift add over raw SQLite? Type-safe generated APIs, query abstractions, streams, and migration support.
2. What is a companion? A typed value object for inserts/partial updates.
3. Why use a stream query? UI can react to relevant database changes.
4. Does code generation remove migration responsibility? No.

# Try It Yourself

1. Add a due-date column and migration.
2. Add a filter for incomplete tasks.
3. Display query results in `StreamBuilder` with loading/error states.
4. Inspect generated table and companion APIs.

# Challenge Project

Design a Drift database for notes and tags with a many-to-many relation, typed search query, reactive list, and tested versioned migration.

# Quick Revision Sheet

- Drift is a type-safe reactive layer over SQLite.
- Declare tables/queries and generate code.
- Streams can notify UI of database changes.
- Schema version and migrations remain your responsibility.
- Configure executor and platform dependencies.

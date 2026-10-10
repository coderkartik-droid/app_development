# Introduction

Local-storage interviews assess data modeling, persistence choice, asynchronous behavior, schema evolution, security, and offline synchronization. Strong answers connect a requirement to a store and explain trade-offs.

# Why this concept exists

Naming a package does not show whether a developer understands data lifetime, privacy, transactions, migrations, and failures. Scenario practice reveals those decisions.

# Real-world analogy

Choosing storage is like choosing containers for a move. The interviewer wants to know what must travel, how fragile it is, who can open it, and how it will be unpacked later.

# Internal Working

Use a decision sequence: identify data shape and size; determine query/transaction needs; classify sensitivity; choose local/cloud/lifetime; define schema and migration; then design error, backup, and sync behavior.

# Syntax

```dart
abstract interface class TaskRepository {
  Future<List<Task>> listOpen({required int limit});
  Future<void> save(Task task);
  Future<void> delete(String id);
}
```

# Complete Example 1

The runnable sample demonstrates the repository boundary frequently discussed in interviews. The implementation is in-memory to keep the code dependency-free; replace it with SQLite/Drift for durable records.

```dart
import 'package:flutter/material.dart';

class Task {
  const Task({required this.id, required this.title});
  final String id;
  final String title;
}

abstract interface class TaskRepository {
  Future<List<Task>> listOpen({required int limit});
  Future<void> save(Task task);
}

class MemoryTaskRepository implements TaskRepository {
  final List<Task> _items = [];

  @override
  Future<List<Task>> listOpen({required int limit}) async =>
      _items.take(limit).toList();

  @override
  Future<void> save(Task task) async {
    _items.removeWhere((item) => item.id == task.id);
    _items.add(task);
  }
}

void main() => runApp(const InterviewStorageApp());

class InterviewStorageApp extends StatelessWidget {
  const InterviewStorageApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: InterviewStoragePage());
}

class InterviewStoragePage extends StatefulWidget {
  const InterviewStoragePage({super.key});

  @override
  State<InterviewStoragePage> createState() => _InterviewStoragePageState();
}

class _InterviewStoragePageState extends State<InterviewStoragePage> {
  final TaskRepository _repository = MemoryTaskRepository();
  List<Task> _items = [];

  Future<void> _add() async {
    final task = Task(id: DateTime.now().microsecondsSinceEpoch.toString(), title: 'Practice answer');
    await _repository.save(task);
    final items = await _repository.listOpen(limit: 20);
    if (mounted) setState(() => _items = items);
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Repository example')),
        body: Column(
          children: [
            FilledButton(onPressed: _add, child: const Text('Add task')),
            Expanded(
              child: ListView(
                children: _items.map((task) => ListTile(title: Text(task.title))).toList(),
              ),
            ),
          ],
        ),
      );
}
```

### Line-by-line explanation

- `Task` is a typed domain model with stable identity.
- `TaskRepository` defines operations independently of storage technology.
- `MemoryTaskRepository` is a fake; it supports tests and examples but loses data on restart.
- `save` replaces a matching ID and appends the new task.
- The page depends on the interface.
- `_add` saves, reloads a bounded list, and updates UI if mounted.
- `ListView` displays typed records.

### Widget tree

```text
MaterialApp → InterviewStoragePage → Scaffold → Column
                                               ├→ Add task
                                               └→ ListView
```

### Execution flow

```text
UI → TaskRepository contract → concrete storage adapter
Save → query latest records → state update → rendered list
```

### ASCII phone preview

```text
+---------------------------+
|    Repository example     |
+---------------------------+
| [       Add task       ]  |
| Practice answer           |
+---------------------------+
```

### Example explanation

The UI is decoupled from the storage engine. A test can inject a fake, while production can inject a database repository. The sample ID generation is demonstration-only; production should use a robust ID strategy.

If the UI directly depends on Hive/SQLite calls, changing persistence affects screens. If the repository is in-memory, it is not durable despite its interface name.

**Common mistakes:** Clearly distinguish test fakes from persistent implementations.

**Professional use case:** Repository abstraction supports testing and store replacement.

# Complete Example 2

A good store-selection answer:

```text
Theme setting → preferences
Relational searchable notes → SQLite/Drift
Refresh token → secure storage
Image bytes → app files, metadata/path in database
```

Explain the reason for each choice, not just the package name.

# Complete Example 3

A migration answer should mention schema versions, deterministic transformations, defaults for new fields, and tests against old-version fixtures.

# Advanced Example

Practice scenario responses using data classification, query patterns, transactional boundaries, sync model, conflict resolution, and account isolation. Explain how errors and partial failures affect the user rather than claiming every store operation is reliable.

# Under the Hood

Storage adapters convert app types to persistent formats. Engines handle files, transactions, indexes, or OS credential APIs. Repositories coordinate those mechanisms and expose application-level operations.

# Performance Notes

- Mention indexes, pagination, batching, and realistic profiling.
- Avoid theoretical performance claims without measurement.
- Distinguish UI-thread work from storage engine execution.

# Best Practices

- State assumptions.
- Choose by requirements and threat model.
- Explain migrations and testing.
- Cover asynchronous errors and lifecycle.
- Separate persistence from synchronization.

# Common Mistakes

- Calling preferences a database.
- Claiming local storage syncs across devices.
- Ignoring schema migration.
- Storing secrets in plain text.
- Overengineering one simple setting.

# Debugging

- Trace data ownership and lifecycle.
- Test cold start, upgrade, and offline paths.
- Verify operation result and schema version.
- Keep diagnostics sanitized.

# Interview Questions

1. Compare preferences, Hive, and SQLite.
2. How do you store refresh tokens safely?
3. How do you migrate a local schema?
4. What is offline-first and how do conflicts work?
5. Why use a repository?
6. How do you prevent SQL injection?
7. What is a cache invalidation policy?
8. Is local persistence a backup?
9. How do you isolate data between accounts?
10. How do you handle partial failure across database and files?

# Try It Yourself

1. Answer each question in under one minute, then add a production trade-off.
2. Design storage for a photo journal.
3. Explain a safe migration.
4. Describe token restoration and sign-out.

# Challenge Project

Prepare a whiteboard design for an offline inventory app. Include data model, local database, images, preferences, secure credentials, sync queue, conflict policy, migration, and testing strategy.

# Quick Revision Sheet

- Preferences: primitive settings.
- Hive: key-value objects with adapters.
- SQLite/Drift: relational data and queries.
- Secure storage: small credentials.
- Files: large binary content.
- Migrations preserve old data.
- Offline-first requires outbox, retry, idempotency, and conflict rules.
- Repository separates UI from storage.

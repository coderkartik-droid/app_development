# Introduction

An offline todo app lets users create and complete tasks without relying on an active network. This chapter builds the user-facing shell and explains how to replace the in-memory demo repository with a durable database and sync queue.

# Why this concept exists

Task capture needs to work immediately. A network outage should not prevent recording a reminder or checking off a task.

# Real-world analogy

A pocket task list remains usable while away from the office. Later, it can be reconciled with a shared team board.

# Internal Working

The UI writes tasks to a local repository and listens for changes. A production offline-first repository stores each task and pending sync operation in one database transaction. A background worker uploads pending operations idempotently and updates sync status.

# Syntax

```dart
class Todo {
  const Todo({required this.id, required this.title, required this.done});
  final String id;
  final String title;
  final bool done;
}
```

# Complete Example 1

This runnable app implements local todo CRUD in memory and visibly marks items “Local demo.” To make it truly offline-persistent, replace the repository with SQLite/Drift or Hive.

```dart
import 'package:flutter/material.dart';

class Todo {
  const Todo({required this.id, required this.title, required this.done});
  final String id;
  final String title;
  final bool done;

  Todo copyWith({bool? done}) =>
      Todo(id: id, title: title, done: done ?? this.done);
}

class TodoRepository {
  final List<Todo> _items = [];
  List<Todo> get items => List.unmodifiable(_items);

  void add(String title) {
    _items.insert(
      0,
      Todo(
        id: DateTime.now().microsecondsSinceEpoch.toString(),
        title: title,
        done: false,
      ),
    );
  }

  void toggle(String id) {
    final index = _items.indexWhere((item) => item.id == id);
    if (index >= 0) _items[index] = _items[index].copyWith(done: !_items[index].done);
  }
}

void main() => runApp(const OfflineTodoApp());

class OfflineTodoApp extends StatelessWidget {
  const OfflineTodoApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: OfflineTodoPage());
}

class OfflineTodoPage extends StatefulWidget {
  const OfflineTodoPage({super.key});

  @override
  State<OfflineTodoPage> createState() => _OfflineTodoPageState();
}

class _OfflineTodoPageState extends State<OfflineTodoPage> {
  final _repository = TodoRepository();
  final _input = TextEditingController();

  void _add() {
    final title = _input.text.trim();
    if (title.isEmpty) return;
    setState(() {
      _repository.add(title);
      _input.clear();
    });
  }

  @override
  void dispose() {
    _input.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Offline todo')),
        body: Column(
          children: [
            const ListTile(
              leading: Icon(Icons.phone_android),
              title: Text('Local demo — not persisted after restart'),
            ),
            TextField(
              controller: _input,
              decoration: const InputDecoration(labelText: 'Task'),
              onSubmitted: (_) => _add(),
            ),
            FilledButton(onPressed: _add, child: const Text('Add task')),
            Expanded(
              child: _repository.items.isEmpty
                  ? const Center(child: Text('No tasks'))
                  : ListView(
                      children: _repository.items.map((todo) {
                        return CheckboxListTile(
                          value: todo.done,
                          title: Text(todo.title),
                          onChanged: (_) => setState(() => _repository.toggle(todo.id)),
                        );
                      }).toList(),
                    ),
            ),
          ],
        ),
      );
}
```

### Line-by-line explanation

- `Todo` is immutable and exposes `copyWith` for a completion update.
- `TodoRepository` owns task records and returns an unmodifiable list.
- `add` generates a demonstration ID and inserts the task at the top.
- `toggle` finds a task by ID and replaces it with a copied completed state.
- `_add` trims and validates input, mutates repository state, and rebuilds.
- `onSubmitted` lets the keyboard action add a task.
- `CheckboxListTile` provides an accessible completion control.
- The UI warns that the repository is memory-only.
- `dispose` releases the text controller.

### Widget tree

```text
OfflineTodoPage → Scaffold → Column
                           ├→ local-demo banner
                           ├→ TextField
                           ├→ Add button
                           └→ Expanded → empty state/ListView → CheckboxListTile
```

### Execution flow

```text
Type task → add to repository → setState → row appears
Toggle checkbox → replace task by ID → setState → completion updates
Production: transaction writes task + outbox operation
```

### ASCII phone preview

```text
+---------------------------+
|        Offline todo       |
+---------------------------+
| Local demo — not persisted|
| Task                      |
| [ Call pharmacy        ]  |
| [       Add task       ]  |
| □ Call pharmacy           |
+---------------------------+
```

### Example explanation

This demo proves the UI interaction but does not survive process death. Production storage should write to a database before reporting durable success. The ID-based toggle avoids depending on row position.

If the repository remains memory-only, all tasks vanish after restart. If `copyWith` is removed, immutable task updates require constructing the entire replacement object manually.

**Common mistakes:** Do not claim “offline-first” durability until a local persistent database and restart test exist.

**Professional use case:** Personal or field task capture with background synchronization.

# Complete Example 2

Store a new task and outbox record in one transaction:

```text
BEGIN
INSERT task (..., syncState='pending')
INSERT outbox (operationId, entityId, payload)
COMMIT
```

If either insert fails, roll back both so data and sync intent cannot diverge.

# Complete Example 3

Retry pending sync with an idempotency key:

```text
send(operation, idempotencyKey: operation.id)
markSynced(operation.id) after server acknowledgement
```

Do not mark a task synced before the server confirms the operation.

# Advanced Example

Add edit/delete operations, tombstones, due dates, filtering, and multi-account scoping. Define whether concurrent edits use last-write-wins or server revisions. Test timeout after server commit but before client acknowledgment.

# Under the Hood

Database observation drives UI updates. The sync worker reads durable outbox rows, sends requests, and records acknowledgments. A retryable operation must be idempotent so network ambiguity does not duplicate tasks.

# Performance Notes

- Use a database-backed list query and pagination for large task sets.
- Index completion/due-date filters.
- Avoid rebuilding the full app for each task update.
- Run sync away from UI frame work.

# Best Practices

- Persist locally first.
- Use stable IDs.
- Store task and outbox atomically.
- Expose pending/synced/error state.
- Define retry, conflict, and delete behavior.
- Keep account data isolated.

# Common Mistakes

- Calling an in-memory demo persistent.
- Syncing without idempotency.
- Dropping deletes before server acknowledgment.
- Leaving the outbox unbounded.
- Treating connectivity as proof of server availability.

# Debugging

- Inspect task and outbox tables together.
- Test offline creation, process restart, then reconnect.
- Simulate timeout after remote commit.
- Verify sync status transitions.
- Test account switching with pending tasks.

# Interview Questions

1. Why write locally before network? Immediate, resilient UX.
2. What should be atomic? Local task mutation and corresponding outbox entry.
3. Why idempotency key? Prevent duplicate effects on retry.
4. What is a tombstone? A retained delete record used until sync confirms removal.
5. Is the sample truly persistent? No, it uses in-memory state.

# Try It Yourself

1. Add delete and undo.
2. Add a `pendingSync` status to the model.
3. Write tests for app restart and queued updates.
4. Decide how two-device title conflicts resolve.

# Challenge Project

Implement a database-backed offline todo app with task CRUD, outbox, sync status, retry, account isolation, and process-restart tests. Use a mock server and document conflict behavior.

# Quick Revision Sheet

- Offline-first requires durable local writes, not just an offline banner.
- Use stable IDs and database transactions.
- Outbox + idempotency supports safe retry.
- Keep delete intent until remote acknowledgment.
- Test offline, restart, timeout, and account switching.

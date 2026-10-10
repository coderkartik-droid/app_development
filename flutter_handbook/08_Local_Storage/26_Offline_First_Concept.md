# Introduction

Offline-first design lets users read and perform important actions without an active network connection. The app stores useful data locally and synchronizes changes when connectivity returns.

# Why this concept exists

Networks are slow, intermittent, expensive, and unavailable in many environments. An app that requires a successful request for every action is fragile even when its backend is healthy.

# Real-world analogy

A field worker writes notes into a durable notebook while away from the office, then reconciles the notebook with the central records when connectivity returns. The local notebook is useful, but disagreements need a rule.

# Internal Working

An offline-first app commonly reads from a local database, writes user changes locally first, and records pending synchronization work. A worker retries operations when online. Sync needs stable IDs, idempotency, ordering, conflict resolution, deletion tombstones, and authentication.

Connectivity status is only a hint: a network interface can exist while the server is unreachable. Treat request outcomes as authoritative.

# Syntax

```text
User action → local transaction (data + pending operation)
            → UI observes local database
            → sync worker uploads when possible
            → server response reconciles local record
```

# Complete Example 1

This runnable sample demonstrates offline-first UI behavior with an in-memory repository. It deliberately labels the repository as in-memory; replace it with SQLite/Drift for persistence and a real sync service for production.

```dart
import 'package:flutter/material.dart';

class Todo {
  const Todo({required this.id, required this.title, required this.pending});
  final int id;
  final String title;
  final bool pending;
}

class OfflineTodoRepository {
  final List<Todo> _todos = [];
  int _nextId = 1;

  List<Todo> get todos => List.unmodifiable(_todos);

  void addOffline(String title) {
    _todos.add(Todo(id: _nextId++, title: title, pending: true));
  }
}

void main() => runApp(const OfflineFirstApp());

class OfflineFirstApp extends StatelessWidget {
  const OfflineFirstApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: OfflineFirstPage());
}

class OfflineFirstPage extends StatefulWidget {
  const OfflineFirstPage({super.key});

  @override
  State<OfflineFirstPage> createState() => _OfflineFirstPageState();
}

class _OfflineFirstPageState extends State<OfflineFirstPage> {
  final _repository = OfflineTodoRepository();
  final _input = TextEditingController();

  void _add() {
    final title = _input.text.trim();
    if (title.isEmpty) return;
    setState(() {
      _repository.addOffline(title);
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
        appBar: AppBar(title: const Text('Offline-first todo')),
        body: Column(
          children: [
            const ListTile(
              leading: Icon(Icons.cloud_off),
              title: Text('Offline demo: changes stay local in memory'),
            ),
            TextField(
              controller: _input,
              decoration: const InputDecoration(labelText: 'New task'),
            ),
            FilledButton(onPressed: _add, child: const Text('Add offline')),
            Expanded(
              child: ListView(
                children: _repository.todos
                    .map((todo) => ListTile(
                          title: Text(todo.title),
                          trailing: const Icon(Icons.cloud_upload_outlined),
                          subtitle: Text(todo.pending ? 'Pending sync' : 'Synced'),
                        ))
                    .toList(),
              ),
            ),
          ],
        ),
      );
}
```

### Line-by-line explanation

- `Todo` includes a stable numeric ID, title, and pending-sync marker.
- The demo repository keeps an in-memory list and marks new items pending.
- `_add` validates the title and updates local state immediately.
- The UI explicitly labels that this demo is memory-only, not durable offline storage.
- Each pending row shows a sync icon and status.
- The text controller is disposed.

### Widget tree

```text
MaterialApp → OfflineFirstPage → Scaffold → Column
                                        ├→ offline status
                                        ├→ TextField
                                        ├→ Add button
                                        └→ Expanded → ListView
```

### Execution flow

```text
User adds task → local repository changes → UI updates immediately
Production extension: durable DB transaction → sync queue → server reconcile
```

### ASCII phone preview

```text
+---------------------------+
|   Offline-first todo      |
+---------------------------+
| [cloud off] Offline demo  |
| New task                  |
| [ Draft report         ]  |
| [      Add offline     ]  |
| Draft report   [upload]   |
| Pending sync              |
+---------------------------+
```

### Example explanation

The local write is authoritative for immediate display in this small demo. It is not persistent because the repository is in memory; closing the process loses data. Production requires a database transaction that writes both the task and its pending operation atomically.

If the pending flag is omitted, the UI cannot distinguish unsynced work. If a sync operation is not idempotent, retrying after a timeout may create duplicate server records.

**Common mistakes:** Do not consider the device's connectivity indicator proof that a particular service is reachable.

**Professional use case:** Field service, task apps, note-taking, and delivery workflows with intermittent network access.

# Complete Example 2

An outbox table can persist operations:

```text
outbox(id, entityId, operation, payload, createdAt, retryCount)
```

Store the domain update and outbox entry in one local transaction so neither can exist without the other.

# Complete Example 3

Use idempotency keys when uploading:

```text
POST /tasks
Idempotency-Key: stable-operation-id
```

If a response is lost and the worker retries, the server can recognize the same operation rather than creating a duplicate.

# Advanced Example

Define conflict policy per entity. Last-write-wins is simple but may discard meaningful edits. Alternatives include server revisions, mergeable fields, user resolution, or append-only events. Represent deletions with tombstones until the server confirms them; otherwise an old remote record can reappear.

# Under the Hood

The local database is observed by the UI. A sync worker reads pending operations, sends authenticated requests, and writes acknowledgments or reconciled server state back locally. Connectivity signals can schedule work, but retries and server results control correctness.

# Performance Notes

- Batch sync requests where API allows.
- Use bounded exponential backoff and avoid retry storms.
- Paginate remote reconciliation.
- Keep pending queue indexes and cleanup policies.
- Do not block UI rendering on background synchronization.

# Best Practices

- Persist local user changes before showing durable success.
- Use stable IDs and idempotent operations.
- Make sync state visible when useful.
- Define conflicts, deletions, retries, and account changes.
- Encrypt sensitive offline data according to threat model.
- Test process death during sync.

# Common Mistakes

- Treating connectivity state as a successful request.
- No conflict resolution.
- Duplicate operations on retry.
- Losing pending work when process exits.
- Resurrecting deleted records.
- Syncing data under the wrong account.

# Debugging

- Inspect local records and outbox entries.
- Simulate airplane mode, timeouts, and response loss.
- Verify retries are idempotent.
- Test sign-out with queued account data.
- Track sync attempt IDs and statuses without logging private payloads.

# Interview Questions

1. What makes an app offline-first? Local data supports primary reading/writing, with later synchronization.
2. Why use an outbox? It durably records pending remote operations.
3. What is idempotency? Repeating an operation has the same effect as performing it once.
4. Why are tombstones useful? They preserve delete intent until remote reconciliation.
5. Is connectivity status authoritative? No; actual request success determines reachability.

# Try It Yourself

1. Add a pending operation record to a local task.
2. Design a retry count and backoff schedule.
3. Simulate a lost server response.
4. Write a conflict policy for simultaneous title edits.

# Challenge Project

Build an offline todo repository with durable tasks, outbox, retry, idempotency keys, visible sync status, and a documented conflict policy. Test killing the app during an in-flight sync.

# Quick Revision Sheet

- Read/write locally first for offline-first flows.
- Persist domain change and outbox atomically.
- Use stable IDs, idempotency, retry, and conflict rules.
- Connectivity is a hint; request outcome is authoritative.
- Preserve delete intent with tombstones.
- Test offline, timeout, retry, restart, and account switching.

# Introduction

CRUD means create, read, update, and delete. Hive boxes provide operations for each action, using a key or an auto-generated index depending on the workflow.

# Why this concept exists

Most local features need more than saving once. Notes, tasks, and bookmarks must be created, listed, edited, and removed while the UI stays synchronized.

# Real-world analogy

CRUD is a filing desk: add a folder, find it, revise its contents, or remove it.

# Internal Working

Box operations update persisted data and notify listeners. Keyed `put` creates or replaces a value; `get` reads; `delete` removes by key. `add` appends and assigns an index-like key. Choose stable keys for records that need identity.

# Syntax

```dart
await box.put('task-1', task);  // create or replace
final task = box.get('task-1'); // read
await box.delete('task-1');    // delete
```

# Complete Example 1

This self-contained UI sample shows CRUD for string records in a Hive box. Add Hive packages and initialize/open the box before `runApp`.

```dart
import 'package:flutter/material.dart';
import 'package:hive_flutter/hive_flutter.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Hive.initFlutter();
  await Hive.openBox<String>('tasks');
  runApp(const HiveCrudApp());
}

class HiveCrudApp extends StatelessWidget {
  const HiveCrudApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: HiveCrudPage());
}

class HiveCrudPage extends StatefulWidget {
  const HiveCrudPage({super.key});

  @override
  State<HiveCrudPage> createState() => _HiveCrudPageState();
}

class _HiveCrudPageState extends State<HiveCrudPage> {
  final _input = TextEditingController();
  late final Box<String> _box;

  @override
  void initState() {
    super.initState();
    _box = Hive.box<String>('tasks');
  }

  Future<void> _create() async {
    final value = _input.text.trim();
    if (value.isEmpty) return;
    await _box.add(value);
    _input.clear();
  }

  Future<void> _update(int index) async {
    final oldValue = _box.getAt(index);
    if (oldValue == null) return;
    await _box.putAt(index, '$oldValue (edited)');
  }

  Future<void> _delete(int index) => _box.deleteAt(index);

  @override
  void dispose() {
    _input.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Hive CRUD')),
        body: Column(
          children: [
            TextField(
              controller: _input,
              decoration: const InputDecoration(labelText: 'Task'),
            ),
            FilledButton(onPressed: _create, child: const Text('Create')),
            Expanded(
              child: ValueListenableBuilder<Box<String>>(
                valueListenable: _box.listenable(),
                builder: (context, box, _) => ListView.builder(
                  itemCount: box.length,
                  itemBuilder: (context, index) => ListTile(
                    title: Text(box.getAt(index) ?? ''),
                    onTap: () => _update(index),
                    trailing: IconButton(
                      icon: const Icon(Icons.delete),
                      onPressed: () => _delete(index),
                    ),
                  ),
                ),
              ),
            ),
          ],
        ),
      );
}
```

### Line-by-line explanation

- Startup initializes Hive and opens the typed task box.
- `_create` validates text and appends a value.
- `_update` reads a record by index, handles null, and replaces it with edited text.
- `_delete` removes the selected index.
- `ValueListenableBuilder` observes box changes and redraws the list.
- `ListView.builder` creates visible rows on demand.
- Tapping a row updates it; the trailing icon deletes it.
- The controller is disposed with state.

### Widget tree

```text
HiveCrudPage → Scaffold → Column
                          ├→ TextField
                          ├→ Create button
                          └→ Expanded → ValueListenableBuilder → ListView
```

### Execution flow

```text
Create → add
Read → list builder reads values
Update → putAt
Delete → deleteAt
Each box change → listener rebuilds list
```

### ASCII phone preview

```text
+---------------------------+
|          Hive CRUD        |
+---------------------------+
| Task                      |
| [ Water plants         ]  |
| [        Create        ]  |
| Water plants         [x]  |
+---------------------------+
```

### Example explanation

The sample uses indexes for compactness, but indexes shift after deletion and should not be treated as durable identity. Production records should use stable keys such as UUIDs or domain IDs.

If the listener is removed, the list may not update after a box operation. If the update null check is removed, a missing value may be force-unwrapped. If the box initialization is omitted, CRUD calls cannot work.

**Common mistakes:** Never use a visible list index as a stable record ID when reorder/removal is possible.

**Professional use case:** Local drafts or task records with reactive UI updates.

# Complete Example 2

Read by stable key:

```dart
final task = box.get(taskId);
```

Return a nullable result to the caller and show a not-found state rather than assuming the record exists.

# Complete Example 3

Watch a single record:

```dart
ValueListenableBuilder(
  valueListenable: box.listenable(keys: [taskId]),
  builder: (context, box, _) => Text(box.get(taskId)?.title ?? 'Deleted'),
)
```

This avoids reacting to unrelated box changes when the widget only needs one keyed record.

# Advanced Example

For multi-step changes, define consistency boundaries and use supported Hive transactions/batches where available. Ensure edits are not lost when another screen deletes the record. Stable keys, repository APIs, and explicit not-found results make these cases manageable.

# Under the Hood

Box mutations update the box's persisted representation and notify listeners. `ValueListenableBuilder` subscribes and rebuilds when notified. Persistence and UI observation are connected but remain separate layers.

# Performance Notes

- Use lazy list widgets for many records.
- Listen to specific keys when possible.
- Avoid repeatedly decoding large nested objects.
- Keep records small and store media in files.

# Best Practices

- Use stable domain keys.
- Put CRUD operations behind a repository.
- Handle missing data.
- Await mutations and expose errors.
- Use typed boxes and reactive listening.

# Common Mistakes

- Treating indexes as stable IDs.
- Opening/closing the box for every operation.
- Ignoring null results.
- Rebuilding an enormous list eagerly.
- Mutating records without persisting replacement.

# Debugging

- Verify box name and type.
- Confirm CRUD future completes and key is correct.
- Inspect listener scope and widget rebuild.
- Test deletion followed by lookup.
- Check errors during adapter decoding for custom objects.

# Interview Questions

1. What does `put` do when a key exists? Replaces its value.
2. What is the difference between `add` and `put`? `add` assigns an entry key; `put` uses the supplied key.
3. Why prefer stable keys over indexes? Indexes can shift after list mutations.
4. How can a UI react to box changes? Use box listenables and a listening widget.

# Try It Yourself

1. Add a confirmation before delete.
2. Replace index identity with a stable ID.
3. Add completion state to a custom task model.
4. Display an empty state.

# Challenge Project

Implement a Hive-backed bookmark manager with stable IDs, search, edit, delete confirmation, and reactive updates. Explain when its queries would justify moving to SQLite.

# Quick Revision Sheet

- CRUD maps to add/put, get, put/update, delete.
- Stable keys are better than UI indexes.
- Box listenables can drive reactive widgets.
- Handle missing values and failures.
- Use repositories and lazy lists for scale.

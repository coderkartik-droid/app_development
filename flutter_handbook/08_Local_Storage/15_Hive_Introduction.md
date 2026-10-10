# Introduction

Hive is a lightweight, fast key-value database for Dart and Flutter. Unlike preferences, it can store application objects in named boxes. It is useful for local structured data when relational joins and complex SQL are not required.

# Why this concept exists

Preferences are designed for small primitive settings. A local app may need many records such as notes, cached objects, or tasks. Hive provides a box-oriented storage model for those records.

# Real-world analogy

A Hive box is like a labeled cabinet drawer. Each record has a key, and the drawer can be opened to read, write, or remove values.

# Internal Working

Hive stores values in local files and serializes them through supported types and adapters. Initialize Hive before opening boxes. For custom Dart classes, register a type adapter before reading or writing those objects.

Hive APIs are asynchronous for opening and writes. The UI should interact through a repository or controller rather than opening boxes throughout widget build methods.

# Syntax

```dart
await Hive.initFlutter();
await Hive.openBox<String>('settings');
final box = Hive.box<String>('settings');
await box.put('theme', 'dark');
```

Package names and setup vary by Hive version and Flutter integration. Add the chosen Hive packages and follow their current platform instructions.

# Complete Example 1

This app stores simple string notes in a Hive box. It uses the Hive Flutter integration package; add compatible `hive` and `hive_flutter` dependencies.

```dart
import 'package:flutter/material.dart';
import 'package:hive_flutter/hive_flutter.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Hive.initFlutter();
  await Hive.openBox<String>('notes');
  runApp(const HiveNotesApp());
}

class HiveNotesApp extends StatelessWidget {
  const HiveNotesApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: HiveNotesPage());
}

class HiveNotesPage extends StatefulWidget {
  const HiveNotesPage({super.key});

  @override
  State<HiveNotesPage> createState() => _HiveNotesPageState();
}

class _HiveNotesPageState extends State<HiveNotesPage> {
  final _input = TextEditingController();
  late final Box<String> _notes;

  @override
  void initState() {
    super.initState();
    _notes = Hive.box<String>('notes');
  }

  Future<void> _add() async {
    final value = _input.text.trim();
    if (value.isEmpty) return;
    await _notes.add(value);
    if (!mounted) return;
    setState(_input.clear);
  }

  @override
  void dispose() {
    _input.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Hive notes')),
        body: Padding(
          padding: const EdgeInsets.all(20),
          child: Column(
            children: [
              TextField(
                controller: _input,
                decoration: const InputDecoration(labelText: 'Note'),
              ),
              FilledButton(onPressed: _add, child: const Text('Add')),
              Expanded(
                child: ValueListenableBuilder<Box<String>>(
                  valueListenable: _notes.listenable(),
                  builder: (context, box, _) => ListView.builder(
                    itemCount: box.length,
                    itemBuilder: (context, index) {
                      final note = box.getAt(index)!;
                      return ListTile(title: Text(note));
                    },
                  ),
                ),
              ),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `WidgetsFlutterBinding.ensureInitialized` prepares Flutter bindings before asynchronous plugin/database initialization.
- `Hive.initFlutter` chooses the platform data directory.
- `openBox<String>` opens or creates a typed box before the app starts.
- `Hive.box` retrieves the already-open box.
- `_add` validates input and asynchronously appends a value.
- `ValueListenableBuilder` listens to box changes and rebuilds the list.
- The builder reads each stored value by index; the sample assumes values are non-null because it only adds strings.
- The text controller is disposed.

### Widget tree

```text
HiveNotesApp → HiveNotesPage → Scaffold → Column
                                        ├→ TextField
                                        ├→ Add button
                                        └→ ValueListenableBuilder → ListView
```

### Execution flow

```text
Initialize Flutter → initialize Hive → open box → run UI
Add note → box.add → box notifies listener → list rebuilds
```

### ASCII phone preview

```text
+---------------------------+
|         Hive notes        |
+---------------------------+
| Note                      |
| [ Call dentist         ]  |
| [          Add         ]  |
| • Call dentist            |
+---------------------------+
```

### Example explanation

The app opens the box before building UI, which avoids attempting to access a box that is not open. `ValueListenableBuilder` makes the list react to box changes without manually reloading after each operation.

If `ensureInitialized` or Hive initialization is omitted, startup storage access may fail. If `openBox` is omitted, `Hive.box` cannot retrieve it. If no listener is used, the stored value can change without this list reflecting it.

**Common mistakes:** Do not close a shared box while active widgets still use it. For custom objects, register adapters before opening/reading typed data.

**Professional use case:** Small offline collections such as drafts or cached records where key-value access is sufficient.

# Complete Example 2

Read one item with a stable key:

```dart
final box = Hive.box<String>('settings');
final language = box.get('language', defaultValue: 'en');
```

The default is returned when the key is absent. Use typed boxes and stable key naming.

# Complete Example 3

Watch a box reactively:

```dart
ValueListenableBuilder(
  valueListenable: box.listenable(keys: ['theme']),
  builder: (context, box, _) => Text(box.get('theme', defaultValue: 'system')),
)
```

Listening to specific keys can avoid rebuilding for unrelated box changes.

# Advanced Example

Choose a schema migration strategy before changing stored object fields. Keep adapters stable and test opening data written by earlier app versions. For related changes, use the database's supported transaction/batch features or stage and recover carefully.

# Under the Hood

Hive serializes typed values into box files, indexes keys for access, and emits change notifications to listeners. Custom adapters define how object fields are encoded and reconstructed.

# Performance Notes

- Open boxes once and reuse them.
- Use lazy boxes for large values when appropriate.
- Avoid storing huge binary assets as ordinary object values.
- Listen to only the keys or data areas a widget needs.
- Measure startup cost from opening many boxes.

# Best Practices

- Initialize before use and register adapters before opening custom-object boxes.
- Centralize box names and access in repositories.
- Use typed boxes and stable keys.
- Plan migrations and backup/export behavior.
- Close boxes only when their owner guarantees no active users.

# Common Mistakes

- Accessing a box before opening it.
- Registering an adapter too late.
- Changing type IDs or field meanings without migration.
- Closing boxes still used by widgets.
- Treating Hive as a relational database.

# Debugging

- Verify initialization path and box name/type.
- Check adapter registration before box access.
- Inspect stored keys and values in development.
- Test upgrades from prior serialized data.
- Look for box closure while listeners remain mounted.

# Interview Questions

1. What is a Hive box? A named key-value collection persisted locally.
2. Why register adapters? To serialize and reconstruct custom Dart objects.
3. What does `ValueListenableBuilder` provide? UI updates when a listened-to box changes.
4. Is Hive relational? No.

# Try It Yourself

1. Add a delete action to each note.
2. Store a setting in a separate typed box.
3. Listen to only one key.
4. Create a migration plan for adding a field to a stored object.

# Challenge Project

Build a local bookmark box with add, remove, and reactive list views. Add a stable key strategy and explain how older app versions' data will be handled.

# Quick Revision Sheet

- Hive is a local key-value object store.
- Initialize and open boxes before using them.
- Register adapters before custom object access.
- Use typed boxes and reactive listeners.
- Plan schema changes and box lifecycle.

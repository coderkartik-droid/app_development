# Introduction

A Hive box is a named collection of key-value entries. A regular box makes values available after it opens; a lazy box loads values when requested, which can be useful when records are large.

# Why this concept exists

Boxes let an app separate data domains and access records by keys without building SQL tables. Multiple named boxes can keep settings, notes, and cache records distinct.

# Real-world analogy

Boxes are labeled storage bins. A regular box puts the bin's contents within easy reach after opening; a lazy box fetches an item from the shelf only when asked.

# Internal Working

Call `Hive.openBox<T>(name)` to open or create a regular box. The returned box supports key-based and index-based operations. `openLazyBox<T>` returns a lazy box whose `get` operation is asynchronous. Keep the box open while consumers use it.

# Syntax

```dart
final Box<String> box = await Hive.openBox<String>('labels');
await box.put('primary', 'Important');
final label = box.get('primary');
```

# Complete Example 1

This app uses a string box for small labels and reactively displays its values. Package setup is required as in the Hive introduction.

```dart
import 'package:flutter/material.dart';
import 'package:hive_flutter/hive_flutter.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Hive.initFlutter();
  await Hive.openBox<String>('labels');
  runApp(const BoxesApp());
}

class BoxesApp extends StatelessWidget {
  const BoxesApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: BoxesPage());
}

class BoxesPage extends StatelessWidget {
  const BoxesPage({super.key});

  @override
  Widget build(BuildContext context) {
    final box = Hive.box<String>('labels');
    return Scaffold(
      appBar: AppBar(title: const Text('Hive boxes')),
      body: ValueListenableBuilder<Box<String>>(
        valueListenable: box.listenable(),
        builder: (context, currentBox, _) {
          if (currentBox.isEmpty) {
            return const Center(child: Text('No labels yet'));
          }
          return ListView(
            children: currentBox.keys.map((key) {
              return ListTile(
                title: Text(currentBox.get(key) ?? '(missing value)'),
                subtitle: Text('Key: $key'),
              );
            }).toList(),
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => box.put('label-${DateTime.now().millisecondsSinceEpoch}', 'New label'),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### Line-by-line explanation

- Startup initializes Hive and opens the `labels` box before `runApp`.
- `BoxesPage` retrieves the existing typed box.
- `ValueListenableBuilder` subscribes to box changes.
- Empty state appears when there are no entries.
- The list iterates keys and safely handles a missing value.
- The floating action button inserts a uniquely named key/value entry.

### Widget tree

```text
MaterialApp → BoxesPage → Scaffold
                         ├→ AppBar
                         ├→ ValueListenableBuilder → Empty state/ListView
                         └→ FloatingActionButton
```

### Execution flow

```text
Open named box → render contents
Add tap → put(key,value) → box emits change → builder redraws list
```

### ASCII phone preview

```text
+---------------------------+
|        Hive boxes         |
+---------------------------+
| Important                 |
| Key: primary              |
| New label                 |
| Key: label-...            |
|                       [+] |
+---------------------------+
```

### Example explanation

The box name and generic type identify the storage collection and value type. The UI observes the box directly, so additions refresh the list. For stable product records, prefer explicit IDs over timestamps generated at button time.

If the box is not opened before `Hive.box`, retrieval fails. If the listener is removed, the page needs another refresh mechanism. If keys are generated non-uniquely, a `put` can overwrite an existing entry.

**Common mistakes:** Do not assume `keys` ordering is a product sort order. Sort explicitly when ordering matters.

**Professional use case:** Keep feature-specific local datasets isolated by box and repository.

# Complete Example 2

A lazy box defers reading a large value:

```dart
final lazyBox = await Hive.openLazyBox<Uint8List>('largeBlobs');
final bytes = await lazyBox.get('image-42');
```

The read is asynchronous. Use lazy boxes where supported by the chosen Hive version and only when values are large enough to justify deferred loading.

# Complete Example 3

Delete one box entry:

```dart
await box.delete('obsolete-key');
```

Prefer targeted deletion. Deleting or closing the whole box has a much larger scope.

# Advanced Example

Separate boxes by ownership and lifecycle, not arbitrarily by every screen. Define who initializes, closes, migrates, and observes each box. A repository can expose typed methods while hiding raw box operations from widgets.

# Under the Hood

Hive maintains a local file-backed representation and in-memory access structures for opened boxes. Regular and lazy boxes differ in when values are loaded. Listeners notify UI consumers when entries change.

# Performance Notes

- Avoid opening many boxes unnecessarily at startup.
- Use lazy access for large values when appropriate.
- Do not build enormous widget lists eagerly; use `ListView.builder`.
- Sort keys/records explicitly when needed.

# Best Practices

- Give boxes clear stable names.
- Use typed values and repositories.
- Open before use and close only at a well-defined application lifecycle boundary.
- Use stable IDs and explicit ordering.
- Keep large media in files and store paths/metadata.

# Common Mistakes

- Accessing unopened or already closed boxes.
- Confusing list index with stable key.
- Depending on unspecified key ordering.
- Storing large media blobs in regular boxes.
- Creating box names dynamically without cleanup policy.

# Debugging

- Check the exact box name, generic type, and initialization path.
- Inspect keys and box length.
- Confirm lazy reads are awaited.
- Verify unique keys before using `put`.
- Test close/reopen behavior.

# Interview Questions

1. What is a regular box? A named key-value store opened for access.
2. How does a lazy box differ? Values are loaded on demand and reads are asynchronous.
3. Why use typed boxes? They make value expectations explicit.
4. Why not rely on key order? It may not match product ordering requirements.

# Try It Yourself

1. Add sorting by stored value.
2. Replace timestamp keys with stable IDs.
3. Add a lazy box example for larger values.
4. Create a repository that hides the box API.

# Challenge Project

Create separate settings and bookmark boxes with clear initialization, typed APIs, sorting, and a documented close/migration lifecycle.

# Quick Revision Sheet

- A box is a named key-value collection.
- Regular and lazy boxes have different loading behavior.
- Open before access; await lazy reads.
- Use stable keys and explicit sorting.
- Keep media in files and metadata in boxes.

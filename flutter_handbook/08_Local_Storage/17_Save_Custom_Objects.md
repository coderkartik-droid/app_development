# Introduction

Custom objects represent application concepts such as notes, products, and tasks. Persisting them requires converting their fields into a storage format and reconstructing the object when loading.

# Why this concept exists

Storing every field as a separate preference is fragile and hard to query. Object databases can persist typed records through adapters, while relational databases map fields to columns.

# Real-world analogy

A custom object is a completed form. The adapter is the form template that explains how each field is written and how to rebuild the record later.

# Internal Working

Hive custom objects require a registered `TypeAdapter<T>`. Each field receives a stable numeric field ID in the serialized representation. When reading, the adapter maps stored field IDs back to constructor values.

The adapter's type ID must be unique within the app, and field IDs must remain stable across releases. Adding fields should include defaults for older records.

# Syntax

```dart
@HiveType(typeId: 1)
class Note {
  @HiveField(0)
  final String title;
}
```

Code generation and annotation package details depend on Hive version. A generated adapter must be built and registered before opening/reading the box.

# Complete Example 1

The following is a complete model-and-adapter illustration for a Hive version using generated adapters. Add compatible Hive, Hive Flutter, annotation, and generator dependencies, then run the project's build runner to create `note.g.dart`.

```dart
import 'package:hive/hive.dart';
part 'note.g.dart';

@HiveType(typeId: 7)
class Note {
  const Note({required this.title, required this.body});

  @HiveField(0)
  final String title;

  @HiveField(1)
  final String body;
}

Future<void> saveNote(Note note) async {
  Hive.registerAdapter(NoteAdapter());
  final box = await Hive.openBox<Note>('notes');
  await box.put('welcome-note', note);
}
```

### Line-by-line explanation

- `hive.dart` provides annotations and box APIs.
- `part` connects the source model to generated adapter code.
- `@HiveType` declares the serialized type and its stable ID.
- `@HiveField` assigns stable numeric identifiers to object fields.
- `Note` is immutable data with required title and body.
- `NoteAdapter` is generated from the annotations and provides serialization code.
- `registerAdapter` makes the decoder available before box access.
- `openBox<Note>` creates or opens a typed box.
- `put` writes the object under a stable key.

### Widget tree

This persistence operation is independent of a particular widget tree. A UI can call `saveNote` from a form submit handler.

### Execution flow

```text
Model + annotations → code generation → adapter
App startup → register adapter → open typed box → put object
```

### ASCII phone preview

```text
+---------------------------+
|         Note editor       |
+---------------------------+
| Title                     |
| Body                      |
| [         Save          ] |
+---------------------------+
```

### Example explanation

The numeric IDs are part of the persisted schema, not cosmetic annotations. Never reuse a type ID for another class or renumber existing fields after data has shipped. If `part` or generated code is missing, compilation fails; if registration is too late, Hive cannot decode stored values.

**Common mistakes:** Do not change field IDs or type IDs casually. Include compatibility defaults when older stored records do not contain newly added fields.

**Professional use case:** Persist a typed offline note or task object in a box.

# Complete Example 2

Update an object by writing the changed instance at the same key:

```dart
await box.put(
  'welcome-note',
  Note(title: 'Updated title', body: 'Existing body'),
);
```

This replaces the object at that key. Read-modify-write should be coordinated if several operations can update the same object.

# Complete Example 3

Keep model serialization separate from UI:

```dart
abstract interface class NoteRepository {
  Future<void> save(String id, Note note);
  Future<Note?> findById(String id);
}
```

The interface lets widgets depend on behavior rather than generated adapter details, and makes tests easier.

# Advanced Example

For schema evolution, adding a field should provide a default when absent. Removing a field should not reuse its old field ID. Renaming a Dart property can retain the same field ID if its meaning is unchanged. Test migration against serialized fixtures from released versions.

# Under the Hood

The adapter writes a type marker and field/value entries into the Hive representation. On reading, it reconstructs a Dart object from the registered field mapping. Code generation reduces hand-written serialization mistakes but does not design migration policy automatically.

# Performance Notes

- Keep objects focused; avoid giant nested object graphs.
- Store large files separately and persist paths/metadata.
- Use lazy boxes for large values where appropriate.
- Avoid repeatedly serializing unchanged values.

# Best Practices

- Reserve unique type IDs.
- Keep field IDs stable forever after release.
- Register adapters once before box access.
- Provide defaults for newly added fields.
- Use repositories and migration tests.

# Common Mistakes

- Forgetting generated adapter output.
- Registering after opening the box.
- Reusing a field/type ID.
- Assuming code generation migrates arbitrary schema changes.
- Storing binary media in every object.

# Debugging

- Confirm build generation ran and `part` filename matches.
- Check adapter registration and type ID uniqueness.
- Reproduce with data serialized by an older app version.
- Inspect null/default handling for fields absent in old records.

# Interview Questions

1. Why does Hive need a type adapter? It maps object fields to/from serialized storage.
2. Why keep field IDs stable? Persisted data relies on them across application versions.
3. What must happen before reading a custom object? Register its adapter and open the box.
4. Does code generation solve migrations? No; compatibility policy and tests are still needed.

# Try It Yourself

1. Add a `createdAt` field with a safe default for older data.
2. Rename a property without changing its field ID.
3. Write a repository interface and fake implementation.
4. Create a migration test fixture.

# Challenge Project

Persist a typed task with ID, title, completion, and creation time. Add an optional due date in a backward-compatible way and test data created before that field existed.

# Quick Revision Sheet

- Custom Hive objects need registered adapters.
- Type and field IDs are persistent schema identifiers.
- Generate adapter code and register before access.
- New fields need old-record defaults.
- Code generation does not replace migration tests.

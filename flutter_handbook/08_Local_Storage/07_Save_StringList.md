# Introduction

A string-list preference stores a short list of strings under one key. It can suit small sets such as recently selected tags or a compact list of dismissed tutorial IDs.

# Why this concept exists

Some settings contain multiple simple text values. A string list avoids manual delimiter encoding, which can fail when values themselves contain delimiters.

# Real-world analogy

It is one labeled index card with a short bullet list, not a filing cabinet with search, relations, and thousands of records.

# Internal Working

`setStringList` writes a list, and `getStringList` returns a nullable list. Treat the returned list as data to validate and copy before in-memory mutation; call the setter to persist changes.

# Syntax

```dart
await prefs.setStringList('recentTags', ['flutter', 'storage']);
final tags = prefs.getStringList('recentTags') ?? <String>[];
```

# Complete Example 1

This app adds a tag to a small persisted list and displays the result. Add `shared_preferences` to dependencies.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const StringListApp());

class StringListApp extends StatelessWidget {
  const StringListApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: TagsPage());
}

class TagsPage extends StatefulWidget {
  const TagsPage({super.key});

  @override
  State<TagsPage> createState() => _TagsPageState();
}

class _TagsPageState extends State<TagsPage> {
  final _input = TextEditingController();
  List<String> _tags = [];
  bool _loading = true;

  @override
  void initState() {
    super.initState();
    _load();
  }

  Future<void> _load() async {
    final prefs = await SharedPreferences.getInstance();
    final stored = prefs.getStringList('favoriteTags') ?? <String>[];
    if (!mounted) return;
    setState(() {
      _tags = List<String>.from(stored);
      _loading = false;
    });
  }

  Future<void> _add() async {
    final tag = _input.text.trim();
    if (tag.isEmpty || _tags.contains(tag)) return;
    final next = [..._tags, tag];
    final prefs = await SharedPreferences.getInstance();
    final ok = await prefs.setStringList('favoriteTags', next);
    if (!mounted) return;
    if (ok) {
      setState(() {
        _tags = next;
        _input.clear();
      });
    }
  }

  @override
  void dispose() {
    _input.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Favorite tags')),
        body: _loading
            ? const Center(child: CircularProgressIndicator())
            : Padding(
                padding: const EdgeInsets.all(20),
                child: Column(
                  children: [
                    TextField(
                      controller: _input,
                      decoration: const InputDecoration(labelText: 'Tag'),
                    ),
                    FilledButton(onPressed: _add, child: const Text('Add tag')),
                    Expanded(
                      child: ListView(
                        children: _tags.map((tag) => ListTile(title: Text(tag))).toList(),
                      ),
                    ),
                  ],
                ),
              ),
      );
}
```

### Line-by-line explanation

- `_tags` is the in-memory list shown in the UI; the preference is the durable copy.
- `_load` reads the nullable list and falls back to an empty list. `List.from` gives this state its own mutable copy.
- `_add` trims the entry and rejects blanks and duplicates.
- A new list is constructed rather than mutating `_tags` before persistence succeeds.
- The setter is awaited; state and input clear only on success.
- `dispose` releases the input controller.
- `Expanded` bounds the `ListView` inside the column.

### Widget tree

```text
MaterialApp → TagsPage → Scaffold → Column
                                  ├→ TextField
                                  ├→ Add button
                                  └→ Expanded → ListView → ListTile*
```

### Execution flow

```text
Startup → getStringList or [] → render tags
Add tap → validate → create next list → await setStringList
                                      └→ success updates UI
```

### ASCII phone preview

```text
+---------------------------+
|       Favorite tags       |
+---------------------------+
| Tag                       |
| [ flutter              ]  |
| [        Add tag       ]  |
| • design                  |
| • flutter                 |
+---------------------------+
```

### Example explanation

The new list is persisted before replacing visible state, so a failed write does not produce a false success. This is appropriate only for a small simple collection.

If the default empty list is removed, null handling is required. If a database-backed collection grows large or needs per-item update/search, a string-list preference becomes awkward and should be replaced.

**Common mistakes:** Avoid concurrent read-modify-write operations; two additions can overwrite one another. For shared or high-volume lists, use a database.

**Professional use case:** A tiny recent-search history or simple list of dismissed hint IDs.

# Complete Example 2

Remove a value from the list and persist a replacement:

```dart
final next = List<String>.from(current)..remove(value);
await prefs.setStringList('favoriteTags', next);
```

Copying prevents accidental mutation of state owned elsewhere. Handle the write result before presenting success.

# Complete Example 3

Replace the whole value:

```dart
await prefs.setStringList('pinnedFilters', ['unread', 'assigned']);
```

This overwrites the previous list under that key. If order and duplicate values matter, define those semantics explicitly.

# Advanced Example

For concurrent callers, put list operations behind one repository and serialize updates. If records need timestamps, metadata, or individual transactions, model each item in a database rather than encoding extra fields into strings.

# Under the Hood

The plugin stores a typed list using platform preference facilities. The operation replaces the value at the key; it is not a transaction across several keys and does not offer database indexing.

# Performance Notes

- Keep the list small.
- Avoid writing repeatedly during rapid interactions.
- Use a database for large, searchable, or frequently updated collections.
- Avoid loading and rewriting an ever-growing list on every addition.

# Best Practices

- Use a stable key and document list semantics.
- Copy before changing state.
- Validate length and duplicates.
- Await persistence before reporting success.
- Keep collection ownership in a repository.

# Common Mistakes

- Treating a string list as a database table.
- Mutating a shared list before the write completes.
- Allowing concurrent additions to overwrite each other.
- Saving empty or duplicate entries unintentionally.
- Forgetting null handling.

# Debugging

- Check that the key contains a list of strings.
- Inspect the list before and after the write.
- Test restart restoration and duplicate handling.
- Look for racing operations from multiple widgets.

# Interview Questions

1. What does `getStringList` return for a missing key? `null`.
2. Why copy a list before editing? To avoid mutating shared/in-memory state prematurely.
3. When should this become a database table? When the collection grows or needs queries, metadata, or robust concurrent updates.
4. Does `setStringList` append? No; it replaces the value for the key.

# Try It Yourself

1. Add a remove button to each tag.
2. Add a maximum list length.
3. Prevent duplicate values regardless of letter case.
4. Simulate a failed write and keep the old list visible.

# Challenge Project

Build a small recent-search preference that keeps only the latest five unique queries, with newest first. Add clear-all behavior and explain when you would migrate it to a database.

# Quick Revision Sheet

- `setStringList` replaces a key's list; `getStringList` is nullable.
- Use for short simple lists only.
- Copy before mutation and await writes.
- Concurrent read-modify-write can lose updates.
- Use a database for querying or growing collections.

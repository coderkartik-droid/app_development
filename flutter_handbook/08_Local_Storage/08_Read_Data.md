# Introduction

Reading persisted data restores a value from the device into the running app. A read has at least three possible outcomes: a valid value, no value, or an error.

# Why this concept exists

An app needs to distinguish first launch from storage failure and from a legitimate empty value. Explicit loading and error states prevent confusing flicker and false defaults.

# Real-world analogy

Opening a labeled drawer can reveal a document, an empty drawer, or a jammed lock. Those outcomes should not all be interpreted as “the document says nothing.”

# Internal Working

Storage APIs are commonly asynchronous. Start the read from an appropriate lifecycle or repository method, show a loading state if needed, and update UI only while its state is mounted. Database reads may return streams or futures depending on the library.

# Syntax

```dart
final prefs = await SharedPreferences.getInstance();
final value = prefs.getString('displayName');
if (value == null) {
  // Key is absent; choose a product-defined first-run behavior.
}
```

# Complete Example 1

The following app reads a persisted note title and presents loading, empty, and error states. Add `shared_preferences` to dependencies.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const ReadDataApp());

class ReadDataApp extends StatelessWidget {
  const ReadDataApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: ReadDataPage());
}

class ReadDataPage extends StatefulWidget {
  const ReadDataPage({super.key});

  @override
  State<ReadDataPage> createState() => _ReadDataPageState();
}

class _ReadDataPageState extends State<ReadDataPage> {
  String? _title;
  bool _loading = true;
  String? _error;

  @override
  void initState() {
    super.initState();
    _readTitle();
  }

  Future<void> _readTitle() async {
    try {
      final prefs = await SharedPreferences.getInstance();
      final title = prefs.getString('draftTitle');
      if (!mounted) return;
      setState(() {
        _title = title;
        _loading = false;
      });
    } catch (error) {
      if (!mounted) return;
      setState(() {
        _loading = false;
        _error = 'Unable to read the saved title.';
      });
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Read persisted data')),
        body: Center(
          child: _loading
              ? const CircularProgressIndicator()
              : _error != null
                  ? Text(_error!)
                  : Text(_title ?? 'No saved draft'),
        ),
      );
}
```

### Line-by-line explanation

- State tracks a nullable title, a loading flag, and a distinct error.
- `initState` begins the read once.
- `_readTitle` awaits store access and reads the nullable value.
- The `mounted` check prevents a late result from updating disposed state.
- The catch branch represents failure separately from missing data.
- The build method renders exactly one of loading, error, stored title, or empty state.

### Widget tree

```text
MaterialApp → ReadDataPage → Scaffold → Center
                                    ├→ loading
                                    ├→ error
                                    ├→ saved title
                                    └→ empty message
```

### Execution flow

```text
Page created → loading=true → async read
             ├→ key found → show title
             ├→ key absent → show empty state
             └→ read throws → show error state
```

### ASCII phone preview

```text
+---------------------------+
|  Read persisted data      |
+---------------------------+
|                           |
|       My draft title      |
|                           |
+---------------------------+
```

### Example explanation

`null` means there is no stored title; an exception means the read failed. Treating both as the same default would hide operational failures. If `mounted` is omitted, an asynchronous completion can update a page that is no longer present.

**Common mistakes:** Avoid doing the same asynchronous read on every `build`. Read in initialization or expose it through a state/repository layer.

**Professional use case:** Restore a draft title while keeping first-run and read-error UX distinct.

# Complete Example 2

Read an integer with a default:

```dart
final savedTab = prefs.getInt('selectedTab') ?? 0;
final safeTab = savedTab.clamp(0, tabCount - 1).toInt();
```

Clamping protects against removed tabs or old values outside the current range. Ensure `tabCount` is positive before calculating the upper bound.

# Complete Example 3

A database read can return a list of records:

```dart
final tasks = await database.select(database.tasks).get();
```

A UI can show a loading state while the future resolves; a reactive database stream can instead emit updates as rows change. Use the query shape needed by the screen, not a full-table read by default.

# Advanced Example

For complex screens, model loading, success-empty, success-with-data, and failure as explicit states rather than unrelated booleans that can contradict each other. Support cancellation or ignore stale results when a user changes filters before a query completes.

# Under the Hood

The storage plugin or database asynchronously retrieves bytes/records and decodes them to Dart values. A widget rebuild is scheduled when state changes; the read itself does not draw UI.

# Performance Notes

- Query only required rows and columns.
- Add database indexes for frequent filters.
- Avoid repeated reads in `build`.
- Paginate large collections.
- Move expensive decoding off the UI isolate when justified.

# Best Practices

- Distinguish missing values from errors.
- Show loading feedback when reads are visible to users.
- Validate data against current app expectations.
- Use mounted/stale-request protection for asynchronous UI work.
- Centralize reads in repositories.

# Common Mistakes

- Treating a failed read as a missing value.
- Updating state after disposal.
- Querying all rows for a small screen.
- Reading during every build.
- Trusting old persisted values without validation.

# Debugging

- Log operation errors without sensitive values.
- Verify store initialization and key/schema.
- Test no-value and corrupted-value paths.
- Check for overlapping queries returning out of order.
- Inspect query count and duration for large data.

# Interview Questions

1. What are common read outcomes? Data, absence, or failure.
2. Why show loading? To distinguish pending from empty and avoid misleading UI.
3. Why guard asynchronous state updates? The widget may have been disposed.
4. What is a stale response? A delayed result for an older request that arrives after newer user input.

# Try It Yourself

1. Add a retry button for the error state.
2. Validate an old tab index.
3. Show empty, loading, and error UI separately.
4. Add a query filter and ignore stale results.

# Challenge Project

Build a local task list loader with separate loading, empty, data, and error states; pagination; and retry. Demonstrate how its UI behaves if a query finishes after leaving the page.

# Quick Revision Sheet

- Read asynchronously and model loading explicitly.
- Missing data differs from an error.
- Validate loaded values.
- Avoid reads in `build`.
- Query only what the UI requires.
- Guard state updates after async gaps.

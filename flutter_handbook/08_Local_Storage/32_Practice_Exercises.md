# Introduction

These exercises turn storage theory into practical decisions. Each one asks you to classify data, select a storage mechanism, implement a small operation, or reason about failure.

# Why this concept exists

Production storage involves more than a successful `write`. Developers must understand type safety, retention, migrations, privacy, offline operation, and recovery.

# Real-world analogy

Training with storage is like practicing emergency procedures before an incident. It is safer to discover that a migration plan is incomplete in a test fixture than after users update.

# Internal Working

For every exercise, trace input → validation → repository → persistence → result → UI. Then test missing data, errors, restart, and older versions. For remote data, also test timeout, retry, and conflict.

# Syntax

```dart
Future<void> saveSetting(String key, bool value) async {
  final prefs = await SharedPreferences.getInstance();
  final saved = await prefs.setBool(key, value);
  if (!saved) throw StateError('Preference write failed');
}
```

# Complete Example 1

This dependency-backed exercise app stores a reading goal. Add `shared_preferences`.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const StoragePracticeApp());

class StoragePracticeApp extends StatelessWidget {
  const StoragePracticeApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: ReadingGoalPage());
}

class ReadingGoalPage extends StatefulWidget {
  const ReadingGoalPage({super.key});

  @override
  State<ReadingGoalPage> createState() => _ReadingGoalPageState();
}

class _ReadingGoalPageState extends State<ReadingGoalPage> {
  int _goal = 10;
  bool _loading = true;

  @override
  void initState() {
    super.initState();
    _load();
  }

  Future<void> _load() async {
    final prefs = await SharedPreferences.getInstance();
    final value = prefs.getInt('readingGoal') ?? 10;
    if (!mounted) return;
    setState(() {
      _goal = value.clamp(1, 50).toInt();
      _loading = false;
    });
  }

  Future<void> _save(int value) async {
    final safe = value.clamp(1, 50).toInt();
    final prefs = await SharedPreferences.getInstance();
    final ok = await prefs.setInt('readingGoal', safe);
    if (!mounted) return;
    if (ok) setState(() => _goal = safe);
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Reading goal')),
        body: Center(
          child: _loading
              ? const CircularProgressIndicator()
              : Column(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    Text('Pages per day: $_goal'),
                    Slider(
                      value: _goal.toDouble(),
                      min: 1,
                      max: 50,
                      divisions: 49,
                      onChanged: (value) => _save(value.round()),
                    ),
                  ],
                ),
        ),
      );
}
```

### Line-by-line explanation

- The page loads the stored integer once from `initState`.
- Missing data uses a default of 10; old out-of-range values are clamped.
- `_save` validates the new range and updates state only if persistence succeeds.
- The slider maps doubles to integer pages with `round`.
- A loading indicator separates initialization from the default.

### Widget tree

```text
MaterialApp → ReadingGoalPage → Scaffold → Center
                                        ├→ loading
                                        └→ Column(Text, Slider)
```

### Execution flow

```text
Read int → default/clamp → slider
Move slider → round → save int → update displayed goal
```

### ASCII phone preview

```text
+---------------------------+
|       Reading goal        |
+---------------------------+
| Pages per day: 20         |
| [--------o----------]      |
+---------------------------+
```

### Example explanation

The persisted value is validated before use, and the widget receives a `double` as required by `Slider`. If `.round()` is removed, the API types no longer match. If clamping is removed, stale data can violate slider bounds.

**Common mistakes:** Decide whether to save continuously or only when dragging ends; continuous writes may be unnecessary.

**Professional use case:** A compact numerical app preference.

# Complete Example 2

Exercise: design a `notes` SQLite table with `id`, `title`, `body`, and `updatedAt`. Write parameterized create, search, and update operations.

### Explanation

Use a primary key and bound user data. Add an index only when a real query pattern needs it; add migrations when the schema changes.

# Complete Example 3

Exercise: use secure storage for a fake refresh-token flow and add clear-on-sign-out behavior.

### Explanation

The secure-storage API protects local token material, but a service must validate it. Tests should never contain production credentials.

# Advanced Example

Add tests for missing preference, old schema migration, failed write, account switch, offline queue retry, and data deletion. Use injected repositories and database fixtures so tests are deterministic.

# Under the Hood

Widget state reflects repository results. Persistent state is serialized and decoded by the selected store. Tests can use fakes for behavior and real temporary databases for SQL/migration behavior.

# Performance Notes

- Exercise realistic record counts.
- Measure query and cold-start time.
- Avoid saving on every high-frequency input event without reason.
- Paginate data in list exercises.

# Best Practices

- Keep exercise scope small.
- State expected behavior before coding.
- Test normal, empty, error, and restart cases.
- Use appropriate store and security boundary.
- Document why a particular storage type is selected.

# Common Mistakes

- Testing only a happy path.
- Using an in-memory fake to claim persistence.
- Ignoring units and value bounds.
- Saving sensitive sample credentials in fixtures.
- Omitting migration and account-isolation scenarios.

# Debugging

- Check repository calls and returned results.
- Cold restart to test persistence.
- Verify type and schema.
- Use sanitized logs and controlled fixtures.

# Interview Questions

1. How would you test persistence across restart?
2. What belongs in preferences versus a database?
3. How would you test a migration?
4. How would you safely retry offline work?
5. How do you avoid exposing secrets in tests?

# Try It Yourself

1. Add a reset action to the reading goal.
2. Write a repository fake that throws on save.
3. Add a schema migration test outline.
4. Classify app data by sensitivity.

# Challenge Project

Build a small offline reading tracker using preferences for display settings and SQLite/Drift for records. Include a repository, query/filter, migration notes, and tests for cold-start loading.

# Quick Revision Sheet

- Validate persisted data.
- Test empty, error, cold start, and migration.
- Fakes test app logic; real temporary DB tests persistence mechanics.
- Match storage to data and sensitivity.
- Measure realistic data volume.

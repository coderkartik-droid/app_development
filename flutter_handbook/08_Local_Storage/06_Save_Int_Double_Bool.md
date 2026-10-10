# Introduction

Preferences can store integers, decimal numbers, and booleans. These types fit small settings such as a selected tab index, a volume level, or whether a hint has been dismissed.

# Why this concept exists

Using a correctly typed value is safer and simpler than encoding a number or flag as text. The type communicates intent and avoids repeated parsing.

# Real-world analogy

A labeled control panel has separate switches, counters, and dials. A boolean is a switch, an integer is a count, and a double is a dial reading.

# Internal Working

Each getter expects the same type written by its setter. A missing key returns `null`. Validate values after reading because old versions, manual test data, or changed requirements may leave a value outside the current supported range.

# Syntax

```dart
await prefs.setInt('launchCount', 4);
await prefs.setDouble('volume', 0.7);
await prefs.setBool('tipsEnabled', true);
```

# Complete Example 1

This settings screen persists a volume level. Add the `shared_preferences` dependency.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const NumericPreferenceApp());

class NumericPreferenceApp extends StatelessWidget {
  const NumericPreferenceApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: VolumePage());
}

class VolumePage extends StatefulWidget {
  const VolumePage({super.key});

  @override
  State<VolumePage> createState() => _VolumePageState();
}

class _VolumePageState extends State<VolumePage> {
  double _volume = 0.5;
  bool _loading = true;

  @override
  void initState() {
    super.initState();
    _load();
  }

  Future<void> _load() async {
    final prefs = await SharedPreferences.getInstance();
    final saved = prefs.getDouble('volume');
    if (!mounted) return;
    setState(() {
      _volume = (saved ?? 0.5).clamp(0.0, 1.0).toDouble();
      _loading = false;
    });
  }

  Future<void> _save(double value) async {
    final safeValue = value.clamp(0.0, 1.0).toDouble();
    final prefs = await SharedPreferences.getInstance();
    final ok = await prefs.setDouble('volume', safeValue);
    if (!mounted) return;
    if (ok) setState(() => _volume = safeValue);
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Numeric preferences')),
        body: Center(
          child: _loading
              ? const CircularProgressIndicator()
              : Column(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    Text('Volume: ${(_volume * 100).round()}%'),
                    Slider(
                      value: _volume,
                      min: 0,
                      max: 1,
                      onChanged: _save,
                    ),
                  ],
                ),
        ),
      );
}
```

### Line-by-line explanation

- The app imports Material UI and preferences.
- `_volume` has a valid initial default; `_loading` marks whether persisted state is available.
- `_load` reads a nullable double and clamps it into the slider's required range.
- `_save` clamps new input, awaits the write, and only then updates local state.
- The `Slider` declares matching `min`, `max`, and `value` bounds.
- The text converts the fraction into a readable percentage.

### Widget tree

```text
MaterialApp → VolumePage → Scaffold → Center
                                   └→ Column
                                      ├→ percentage Text
                                      └→ Slider
```

### Execution flow

```text
Launch → read double → validate range → render slider
Drag slider → clamp → persist → update displayed percentage
```

### ASCII phone preview

```text
+---------------------------+
| Numeric preferences       |
+---------------------------+
|         Volume: 70%       |
|  [---------o---------]    |
+---------------------------+
```

### Example explanation

The value is clamped because `Slider.value` must be within its bounds. If clamping is removed, stale persisted values can cause an assertion. Saving every pointer movement is simple but may create frequent writes; a production slider may save on interaction completion or debounce writes.

**Common mistakes:** Do not confuse `int` and `double`; the getter must match the stored type.

**Professional use case:** Persisting a sound level, selected tab, or feature toggle.

# Complete Example 2

Store a boolean:

```dart
await prefs.setBool('notificationsEnabled', false);
final enabled = prefs.getBool('notificationsEnabled') ?? true;
```

Choose the default based on product policy and platform permission state; this boolean does not itself grant notification permission.

# Complete Example 3

Store a counter:

```dart
final current = prefs.getInt('launchCount') ?? 0;
await prefs.setInt('launchCount', current + 1);
```

For highly concurrent or transactional counters, use a database; preference read-modify-write is not a general atomic transaction.

# Advanced Example

Version a setting when its meaning changes. For example, if a stored number changes from 0–100 to 0.0–1.0, migrate with explicit conversion, validation, and a version marker rather than guessing each startup.

# Under the Hood

Typed setters encode primitive values through the plugin's platform storage. Type identity is part of the API contract. These operations do not provide multi-key transactions; coordinate related values through a repository or database if atomicity matters.

# Performance Notes

- Avoid writing slider values on every high-frequency drag event if unnecessary.
- Batch or debounce non-critical settings.
- Do not use preferences as an event counter under heavy concurrent updates.
- Validate every loaded value.

# Best Practices

- Match getter/setter types.
- Clamp numeric ranges.
- Choose meaningful defaults.
- Document units (fraction, seconds, points).
- Use transaction-capable storage for related or high-integrity data.

# Common Mistakes

- Reading an int with `getDouble` or vice versa.
- Applying stale out-of-range data directly to a widget.
- Assuming boolean preferences equal OS permissions.
- Performing non-atomic read-modify-write in concurrent workflows.
- Writing on every movement without considering persistence load.

# Debugging

- Inspect both stored type and value.
- Verify min/max match the widget's accepted range.
- Log units and converted values, not just raw numbers.
- Test missing and legacy out-of-range values.

# Interview Questions

1. Why use typed preference values? They avoid parsing and express the setting's intended type.
2. Why clamp loaded values? Persisted values may no longer match current constraints.
3. Is a preference increment atomic? Not generally; use a transaction-capable store for strong guarantees.
4. Does a notification preference grant OS permission? No; permission is managed separately.

# Try It Yourself

1. Persist a selected tab index and clamp it to available tabs.
2. Add a boolean hint preference with a deliberate default.
3. Save a slider value on drag completion instead of every update.
4. Define units for a persisted timeout.

# Challenge Project

Create an app settings model with theme bool, text scale double, and launch count int. Validate all values, migrate one changed range, and test defaults and corrupt/outdated values.

# Quick Revision Sheet

- Use `setInt/getInt`, `setDouble/getDouble`, `setBool/getBool`.
- Match types exactly.
- Validate and clamp values before applying.
- Preferences do not provide general transactions.
- Debounce high-frequency changes.

# Introduction

Debouncing delays an action until a stream of rapid events pauses. It is often used for search fields so typing several characters produces one API request rather than one request per character.

# Why this concept exists

Text input can change many times per second. Sending every intermediate value wastes bandwidth, increases server load, and returns many results the user does not need.

# Real-world analogy

A call center waits briefly after the last digit is entered before dialing. If another digit arrives, the timer resets.

# Internal Working

Each input event cancels/replaces a timer. When the quiet interval passes, the latest value is submitted. Debouncing reduces calls but does not guarantee old in-flight requests are canceled; use cancellation or request IDs for stale-result protection.

# Syntax

```dart
Timer? timer;
timer?.cancel();
timer = Timer(const Duration(milliseconds: 350), runSearch);
```

# Complete Example 1

This runnable app debounces calls to a fake search repository and updates results.

```dart
import 'dart:async';
import 'package:flutter/material.dart';

class SearchRepository {
  Future<List<String>> search(String query) async {
    await Future<void>.delayed(const Duration(milliseconds: 250));
    return ['Result for "$query"', 'More about "$query"'];
  }
}

void main() => runApp(const DebounceApp());

class DebounceApp extends StatelessWidget {
  const DebounceApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: DebouncePage());
}

class DebouncePage extends StatefulWidget {
  const DebouncePage({super.key});

  @override
  State<DebouncePage> createState() => _DebouncePageState();
}

class _DebouncePageState extends State<DebouncePage> {
  final _input = TextEditingController();
  final _repository = SearchRepository();
  Timer? _timer;
  List<String> _results = [];
  bool _loading = false;
  int _version = 0;

  void _onChanged(String value) {
    _timer?.cancel();
    final version = ++_version;
    final query = value.trim();
    if (query.isEmpty) {
      setState(() {
        _results = [];
        _loading = false;
      });
      return;
    }
    _timer = Timer(const Duration(milliseconds: 350), () {
      _runSearch(query, version);
    });
  }

  Future<void> _runSearch(String query, int version) async {
    setState(() => _loading = true);
    final results = await _repository.search(query);
    if (!mounted || version != _version) return;
    setState(() {
      _results = results;
      _loading = false;
    });
  }

  @override
  void dispose() {
    _timer?.cancel();
    _input.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Debounced search')),
        body: Column(
          children: [
            TextField(
              controller: _input,
              onChanged: _onChanged,
              decoration: const InputDecoration(labelText: 'Type to search'),
            ),
            if (_loading) const LinearProgressIndicator(),
            Expanded(
              child: ListView(
                children: _results.map((result) => ListTile(title: Text(result))).toList(),
              ),
            ),
          ],
        ),
      );
}
```

### Line-by-line explanation

- `Timer` from `dart:async` implements the quiet period.
- `_onChanged` cancels the previous timer and increments request identity.
- Empty text clears results and skips the request.
- A 350 ms timer starts `_runSearch` with the latest query/version.
- `_runSearch` shows progress and awaits the repository.
- A stale result or disposed page is ignored.
- `dispose` cancels the timer and disposes input state.

### Widget tree

```text
DebouncePage → Scaffold → Column
                         ├→ TextField
                         ├→ optional progress
                         └→ Expanded ListView
```

### Execution flow

```text
Typing event -> cancel old timer -> start 350ms timer
More typing -> reset timer
Quiet for 350ms -> issue latest query -> render current result
```

### ASCII UI preview

```text
+---------------------------+
|    Debounced search       |
+---------------------------+
| Type to search            |
| [ flutter              ]  |
| [==== loading ========]   |
| • Result for "flutter"    |
+---------------------------+
```

### Example explanation

Debounce waits after typing stops; throttle instead limits action to a maximum rate. The version counter protects against a previous search completing after a newer input event. In production, also handle repository errors and cancellation.

If timer cancellation is removed, every character may trigger a search. If the timer is not canceled in `dispose`, it can call into a page that no longer exists.

**Common mistakes:** Choosing an excessively long debounce makes search feel sluggish; tune using actual UX and API limits.

# Complete Example 2

Debounce button actions such as repeated refresh taps:

```text
ignore refresh if a request is already active
```

For operations where every event matters (payments, audit events), debouncing is not an appropriate substitute for correct queueing.

# Complete Example 3

Throttle scroll analytics to at most one event per interval. This differs from debounce, which waits for inactivity.

# Advanced Example

Combine debounce with cancellation tokens or request IDs, minimum query length, pagination reset, and query-scoped loading/errors. Keep the current results visible or clear them according to UX policy.

# Under the Hood

The Timer schedules a callback on Dart's event loop after a duration. Canceling prevents the callback if it has not fired; it does not cancel a network request already started by that callback.

# Performance Notes

- Tune debounce interval.
- Avoid creating expensive timers per frame/scroll tick.
- Cancel timers on disposal.
- Use request cancellation when supported.

# Best Practices

- Debounce only events where intermediate states are disposable.
- Protect against stale completions.
- Reset paging/cache identity on new query.
- Provide error handling and accessible feedback.

# Common Mistakes

- Confusing debounce with throttle.
- Forgetting timer cancellation.
- Assuming canceling timer cancels HTTP.
- Debouncing irreversible actions.

# Debugging

- Log query count and timer start/fire.
- Test rapid input and pause intervals.
- Simulate delayed responses out of order.
- Confirm empty input cancels pending search.

# Interview Questions

1. What is debounce? Run after a quiet interval following the latest event.
2. How differs from throttle? Throttle caps frequency; debounce waits for inactivity.
3. Does canceling a timer cancel an in-flight HTTP request? No.
4. When not to debounce? When every operation/event must be preserved.

# Try It Yourself

1. Tune delay from 150 to 600 ms.
2. Add clear input while a timer is pending.
3. Add network request cancellation.
4. Compare debounce and throttle for scroll events.

# Challenge Project

Build a live search screen with debounce, cancellation/stale response protection, empty query behavior, pagination reset, and request-count tests.

# Quick Revision Sheet

- Debounce waits after latest event.
- Cancel/reset timer on new input.
- Cancel timer during dispose.
- Does not cancel work already started.
- Combine with stale-result protection.

# Introduction

These exercises help practice API construction, models, errors, pagination, and architecture. Each task starts small and adds a production concern.

# Why this concept exists

Networking knowledge becomes useful when applied to realistic failure and state transitions, not just successful requests.

# Real-world analogy

Practice requests are flight simulations: they let you encounter an engine failure safely before carrying passengers.

# Internal Working

For each exercise trace input → URI/request → response → validation → model → state → UI. Then add at least one failure case, a test, and a security review.

# Syntax

```dart
final uri = Uri.https(host, path, queryParameters);
```

# Complete Example 1

This runnable app fetches a demo list with loading, retry, and empty-state behavior. Add `http`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const ApiPracticeApp());

class ApiPracticeApp extends StatelessWidget {
  const ApiPracticeApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: PracticePage());
}

class PracticePage extends StatefulWidget {
  const PracticePage({super.key});

  @override
  State<PracticePage> createState() => _PracticePageState();
}

class _PracticePageState extends State<PracticePage> {
  List<String> _titles = [];
  bool _loading = false;
  String? _error;

  Future<void> _load() async {
    setState(() {
      _loading = true;
      _error = null;
    });
    try {
      final response = await http.get(
        Uri.https('jsonplaceholder.typicode.com', '/posts'),
      );
      if (response.statusCode != 200) throw Exception('HTTP ${response.statusCode}');
      final rows = jsonDecode(response.body) as List<dynamic>;
      final titles = rows
          .take(5)
          .map((row) => (row as Map<String, dynamic>)['title'] as String)
          .toList();
      if (!mounted) return;
      setState(() => _titles = titles);
    } catch (error) {
      if (!mounted) return;
      setState(() => _error = 'Could not load; try again.');
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('API practice')),
        body: _loading
            ? const Center(child: CircularProgressIndicator())
            : _error != null
                ? Center(
                    child: Column(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        Text(_error!),
                        FilledButton(onPressed: _load, child: const Text('Retry')),
                      ],
                    ),
                  )
                : _titles.isEmpty
                    ? Center(child: FilledButton(onPressed: _load, child: const Text('Load')))
                    : ListView(
                        children: _titles.map((title) => ListTile(title: Text(title))).toList(),
                      ),
      );
}
```

### Line-by-line explanation

- `_load` sets loading and clears prior error.
- GET retrieves a collection, status is validated, JSON is decoded, and five titles are mapped.
- `mounted` guards state updates.
- The UI shows loading, retryable error, initial/empty action, or a result list.
- `finally` resets loading even when request or parsing fails.

### Widget tree

```text
Scaffold → loading / error+retry / initial button / ListView
```

### Execution flow

```text
Load -> GET -> status -> JSON list -> typed strings -> UI
               \-> error -> retry state
```

### ASCII UI preview

```text
+---------------------------+
|       API practice        |
+---------------------------+
| • sunt aut facere...      |
| • qui est esse...         |
+---------------------------+
```

### Example explanation

The exercise uses a real public demo API but caps the display size. Production would use a typed model, repository, timeout, and pagination. This pattern distinguishes pending, failure, and result states.

If `finally` is removed, loading can become stuck. If the request is moved into `build`, rebuilding repeats it.

**Common mistakes:** Do not catch parsing and transport errors indistinguishably in production diagnostics.

# Complete Example 2

Exercise: encode a search term safely:

```dart
Uri.https('api.example.com', '/search', {'q': query});
```

Test spaces, `&`, and Unicode.

# Complete Example 3

Exercise: map status codes 401, 404, and 429 to distinct domain errors. Include a bounded retry for 429 only when allowed by server guidance.

# Advanced Example

Add tests for:

- 200 with valid JSON.
- 204 with no body.
- 401 expired session.
- malformed JSON.
- timeout and offline.
- duplicate search request and stale result.
- pagination end and retry.

# Under the Hood

The request uses asynchronous transport; JSON parsing and mapping convert wire bytes to app values. State updates trigger widget rebuilds. A mock client controls each response deterministically.

# Performance Notes

- Limit response size.
- Use pagination.
- Avoid duplicate requests.
- Profile large model parsing.

# Best Practices

- Start with explicit endpoint contract.
- Test failure and edge states.
- Inject clients/repositories.
- Redact secrets in logs.
- Keep examples separate from production credentials.

# Common Mistakes

- Testing only one successful response.
- No timeout/retry policy.
- Force-casting arbitrary response.
- Confusing fake/in-memory data with a real API.

# Debugging

- Log sanitized method/path/status.
- Compare JSON fixture to model.
- Test cold and network-failure cases.
- Verify loading resets.

# Interview Questions

1. How do you test networking deterministically?
2. Why should the API client be injected?
3. How handle an empty 204?
4. What protects search results from reordering?

# Try It Yourself

1. Add typed response model.
2. Add timeout.
3. Write a mock-client test.
4. Add next-page state.

# Challenge Project

Build a searchable paginated catalog with repository, typed DTO, loading/error/empty states, debounce, and mocked tests.

# Quick Revision Sheet

- Trace request through decode and UI.
- Test success, empty, HTTP error, parse error, and timeout.
- Inject dependencies.
- Paginate and prevent stale results.
- Redact credentials and personal data.

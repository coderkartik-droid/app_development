# Introduction

A search API accepts a query and returns matching results. Search may also support filters, sorting, pagination, and suggestions.

# Why this concept exists

Large datasets should be searched on the server rather than downloaded in full and filtered on a phone. Server search can use indexes, permissions, and current data.

# Real-world analogy

Instead of reading every book in a library, tell the librarian a phrase and receive matching titles.

# Internal Working

The client collects query input, debounces rapid edits, builds encoded query parameters, sends a request, and maps results. It should ignore stale responses when a later query has already replaced them.

# Syntax

```dart
final uri = Uri.https('api.example.com', '/search', {'q': query});
```

# Complete Example 1

This app searches a demo endpoint when the user submits text. Add `http`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const SearchApiApp());

class SearchApiApp extends StatelessWidget {
  const SearchApiApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: SearchApiPage());
}

class SearchApiPage extends StatefulWidget {
  const SearchApiPage({super.key});

  @override
  State<SearchApiPage> createState() => _SearchApiPageState();
}

class _SearchApiPageState extends State<SearchApiPage> {
  final _query = TextEditingController();
  List<String> _results = [];
  bool _loading = false;
  String? _error;
  int _requestVersion = 0;

  Future<void> _search() async {
    final query = _query.text.trim();
    final version = ++_requestVersion;
    if (query.isEmpty) {
      setState(() {
        _results = [];
        _error = null;
      });
      return;
    }
    setState(() {
      _loading = true;
      _error = null;
    });
    try {
      final uri = Uri.https(
        'jsonplaceholder.typicode.com',
        '/posts',
        {'title_like': query},
      );
      final response = await http.get(uri);
      if (response.statusCode != 200) throw Exception('HTTP ${response.statusCode}');
      final rows = jsonDecode(response.body) as List<dynamic>;
      final results = rows
          .map((row) => (row as Map<String, dynamic>)['title'] as String)
          .toList();
      if (!mounted || version != _requestVersion) return;
      setState(() => _results = results);
    } catch (error) {
      if (!mounted || version != _requestVersion) return;
      setState(() => _error = 'Search failed');
    } finally {
      if (mounted && version == _requestVersion) {
        setState(() => _loading = false);
      }
    }
  }

  @override
  void dispose() {
    _query.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Search API')),
        body: Column(
          children: [
            TextField(
              controller: _query,
              decoration: const InputDecoration(labelText: 'Search posts'),
              onSubmitted: (_) => _search(),
            ),
            FilledButton(onPressed: _search, child: const Text('Search')),
            if (_loading) const LinearProgressIndicator(),
            if (_error != null) Text(_error!),
            Expanded(
              child: ListView(
                children: _results.map((item) => ListTile(title: Text(item))).toList(),
              ),
            ),
          ],
        ),
      );
}
```

### Line-by-line explanation

- `_requestVersion` gives each search a monotonically increasing identity.
- Empty input clears results without sending a request.
- The URI builder encodes the query parameter.
- Response is checked and mapped to titles.
- Before applying results/errors, the code confirms this is still the latest request.
- `finally` clears loading only for the latest request, avoiding an old call hiding a newer spinner.
- `TextField.onSubmitted` and button both trigger search.
- The controller is disposed.

### Widget tree

```text
Scaffold → Column
          ├→ TextField
          ├→ Search button
          ├→ progress/error
          └→ Expanded ListView
```

### Execution flow

```text
Query q1 -> request v1
Query q2 -> request v2
Response v1 arrives late -> ignored
Response v2 arrives -> displayed
```

### ASCII UI preview

```text
+---------------------------+
|        Search API         |
+---------------------------+
| Search posts              |
| [ flutter              ]  |
| [        Search        ]  |
| • sunt aut facere...      |
+---------------------------+
```

### Example explanation

Server-side search avoids downloading every post. The version guard prevents an older response from overwriting a newer query. It does not cancel the old request; cancellation can reduce resource use when supported.

If request versioning is removed, results can appear for a query no longer shown in the field. If URI encoding is bypassed, punctuation may alter query structure.

**Common mistakes:** Search matching and ranking should be server-defined; client-side filtering a small result subset can be misleading.

# Complete Example 2

Add filters and pagination:

```dart
{'q': query, 'category': category, 'cursor': cursor}
```

Keep search query and cursor together; a new query must reset pagination.

# Complete Example 3

Show suggestions through a separate endpoint, but cache/debounce carefully. Do not submit every keystroke without rate limits.

# Advanced Example

Handle minimum query length, accent/case normalization, privacy, safe query logging, ranking, and stale-result cancellation. Keep result pagination tied to an immutable query snapshot.

# Under the Hood

The server tokenizes/indexes data and applies ranking/filtering. The client sends encoded query text and maps a result set. Network response order can differ from request order, so request identity matters.

# Performance Notes

- Debounce inputs.
- Use result pagination.
- Cancel obsolete requests where possible.
- Avoid rebuilding entire screen for every character.

# Best Practices

- Encode query parameters.
- Reset cursor on query change.
- Protect against stale responses.
- Provide empty and error states.
- Avoid logging sensitive searches.

# Common Mistakes

- Filtering full datasets on device unnecessarily.
- Stale response overwrite.
- Keeping old cursor for new query.
- Too many requests per second.
- Treating no matches as an error.

# Debugging

- Log query hash/length and request version rather than private query text where appropriate.
- Verify API filters/ranking.
- Reproduce out-of-order responses.
- Check query encoding.

# Interview Questions

1. Why server-side search? Scale, freshness, permissions, indexing.
2. How prevent stale results? Request identity/cancellation.
3. Why reset pagination after query change? Cursor belongs to prior result set.
4. How reduce request volume? Debouncing.

# Try It Yourself

1. Add clear-query behavior.
2. Add search error retry.
3. Add query-specific cursor state.
4. Test responses arriving out of order.

# Challenge Project

Build a product search with debounce, filters, cursor pagination, stale-response protection, empty states, and private-query logging policy.

# Quick Revision Sheet

- Search via encoded query parameters.
- Server owns matching/ranking for large datasets.
- Debounce and paginate.
- Ignore/cancel stale requests.
- Reset paging on new query.

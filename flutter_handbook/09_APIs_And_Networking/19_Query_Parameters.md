# Introduction

Query parameters are key-value options appended to a URI. They are commonly used for filters, search terms, pagination, sorting, and feature flags.

# Why this concept exists

One endpoint can return different subsets or orderings based on parameters while preserving a stable resource path.

# Real-world analogy

Query parameters are extra instructions on a request ticket: “show active orders, newest first, page three.”

# Internal Working

URI builders percent-encode parameter values. The server parses the query and validates each value. Query parameters can appear in logs, browser history, proxy traces, and analytics; do not put secrets in them.

# Syntax

```dart
final uri = Uri.https('api.example.com', '/items', {
  'q': search,
  'limit': '20',
});
```

# Complete Example 1

This runnable app takes a search term and fetches a filtered list of demo posts. Add `http`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const QueryParametersApp());

class QueryParametersApp extends StatelessWidget {
  const QueryParametersApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: QueryParametersPage());
}

class QueryParametersPage extends StatefulWidget {
  const QueryParametersPage({super.key});

  @override
  State<QueryParametersPage> createState() => _QueryParametersPageState();
}

class _QueryParametersPageState extends State<QueryParametersPage> {
  final _controller = TextEditingController(text: 'qui');
  List<String> _titles = [];
  bool _loading = false;
  String? _error;

  Future<void> _search() async {
    setState(() {
      _loading = true;
      _error = null;
    });
    try {
      final uri = Uri.https(
        'jsonplaceholder.typicode.com',
        '/posts',
        {'title_like': _controller.text.trim()},
      );
      final response = await http.get(uri);
      if (response.statusCode != 200) {
        throw Exception('HTTP ${response.statusCode}');
      }
      final data = jsonDecode(response.body) as List<dynamic>;
      final titles = data
          .map((item) => (item as Map<String, dynamic>)['title'] as String)
          .toList();
      if (!mounted) return;
      setState(() => _titles = titles);
    } catch (error) {
      if (!mounted) return;
      setState(() => _error = 'Search failed');
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Query parameters')),
        body: Column(
          children: [
            TextField(
              controller: _controller,
              decoration: const InputDecoration(labelText: 'Title contains'),
              onSubmitted: (_) => _search(),
            ),
            FilledButton(
              onPressed: _loading ? null : _search,
              child: const Text('Search'),
            ),
            if (_loading) const LinearProgressIndicator(),
            if (_error != null) Text(_error!),
            Expanded(
              child: _titles.isEmpty && !_loading && _error == null
                  ? const Center(child: Text('No results yet'))
                  : ListView(
                      children: _titles.map((title) => ListTile(title: Text(title))).toList(),
                    ),
            ),
          ],
        ),
      );
}
```

### Line-by-line explanation

- The controller stores the search string.
- `_search` marks loading, clears previous error, and builds a URI query map.
- `Uri.https` encodes special characters in the search term.
- The response is checked and parsed as a list of post maps.
- `finally` always stops the progress indicator.
- `TextField.onSubmitted` supports keyboard search.
- `Expanded` bounds result list and displays an initial empty state.
- The controller is disposed.

### Widget tree

```text
Scaffold → Column
          ├→ TextField
          ├→ Search button
          ├→ optional progress/error
          └→ Expanded → empty state/ListView
```

### Execution flow

```text
Search text -> query map -> encoded URI -> GET
            -> decode -> titles -> results list
```

### ASCII UI preview

```text
+---------------------------+
|    Query parameters       |
+---------------------------+
| Title contains            |
| [ qui                  ]  |
| [        Search        ]  |
| • qui est esse...         |
+---------------------------+
```

### Example explanation

The query key is API-specific. The app uses URI construction rather than concatenation so spaces and punctuation are encoded correctly.

If query text is concatenated directly, values containing `&` can add unintended parameters. If old results remain during a new request, consider whether they should be cleared, labeled, or shown as stale.

**Common mistakes:** Query parameters are not confidential; URLs can be logged.

# Complete Example 2

Omit an optional filter when it is empty:

```dart
final params = <String, String>{
  'page': '$page',
  if (category != null) 'category': category,
};
```

This avoids sending accidental literal values such as `"null"`.

# Complete Example 3

Repeated keys may be required by some APIs. Check whether the server expects `tag=a&tag=b` or a comma-separated value; URI query-map APIs may need a multi-value representation.

# Advanced Example

Include query parameters in cache keys and stale-request identity. Normalize equivalent query states consistently. Validate page limits and allowed sort fields at both client and server.

# Under the Hood

The URI encoder escapes reserved characters in query values; the server decodes them before applying filters. Query order is generally not semantically important, though signatures/canonicalization may impose rules.

# Performance Notes

- Debounce rapidly changing search terms.
- Use server filters and pagination.
- Cancel or ignore obsolete requests.
- Avoid returning oversized filtered payloads.

# Best Practices

- Build URIs structurally.
- Validate values and defaults.
- Keep credentials in headers.
- Keep query state aligned with cache state.
- Provide empty results and retry UX.

# Common Mistakes

- Manual string concatenation.
- Literal `"null"` values.
- Missing query in cache key.
- Applying stale results.
- Sending sensitive data in URL.

# Debugging

- Inspect final URI with sanitized values.
- Confirm server parameter spelling/casing.
- Test spaces, Unicode, `&`, and `?`.
- Check repeated-parameter conventions.

# Interview Questions

1. What are query parameters used for? Filters and options on a resource request.
2. How do you encode them? Build a `Uri` with query parameters.
3. Why not send secrets in query? URLs are commonly logged and shared.
4. What is stale response protection? Only applying results that match the latest query.

# Try It Yourself

1. Add category and sort parameters.
2. Test query text with Unicode and punctuation.
3. Add pagination.
4. Add cache key construction.

# Challenge Project

Build a search repository that handles filters, optional values, debounce, pagination, cache key, and stale results.

# Quick Revision Sheet

- Query values are URI options.
- Use `Uri` to encode.
- Validate and omit absent parameters.
- URLs are not secret storage.
- Include query in cache and request identity.

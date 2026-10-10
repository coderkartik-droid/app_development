# Introduction

A parameterized GET request adds filters or options to a URL, commonly as query parameters. Examples include search text, page number, sort order, and date range.

# Why this concept exists

One collection endpoint can serve many views without creating a separate URL for every filter combination.

# Real-world analogy

Asking a librarian for “books” is broad; adding “author=Le Guin, sort=title” narrows the search.

# Internal Working

Query parameters follow `?` in a URI and are encoded by the URI builder. The server reads them and applies filtering or pagination. Parameter names, allowed values, and repeated-value semantics are defined by the API.

# Syntax

```dart
final uri = Uri.https(
  'api.example.com',
  '/v1/products',
  {'category': 'books', 'page': '2'},
);
```

# Complete Example 1

This app searches a public posts endpoint by user ID. Add `http`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const QueryGetApp());

class QueryGetApp extends StatelessWidget {
  const QueryGetApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: QueryGetPage());
}

class QueryGetPage extends StatefulWidget {
  const QueryGetPage({super.key});

  @override
  State<QueryGetPage> createState() => _QueryGetPageState();
}

class _QueryGetPageState extends State<QueryGetPage> {
  final _userId = TextEditingController(text: '1');
  List<String> _titles = [];
  String? _error;
  bool _loading = false;

  Future<void> _search() async {
    setState(() {
      _loading = true;
      _error = null;
    });
    try {
      final uri = Uri.https(
        'jsonplaceholder.typicode.com',
        '/posts',
        {'userId': _userId.text.trim()},
      );
      final response = await http.get(uri);
      if (response.statusCode != 200) {
        throw Exception('HTTP ${response.statusCode}');
      }
      final rows = jsonDecode(response.body) as List<dynamic>;
      final titles = rows
          .map((row) => (row as Map<String, dynamic>)['title'] as String)
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
    _userId.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('GET with query')),
        body: Column(
          children: [
            TextField(
              controller: _userId,
              keyboardType: TextInputType.number,
              decoration: const InputDecoration(labelText: 'User ID'),
            ),
            FilledButton(
              onPressed: _loading ? null : _search,
              child: const Text('Search'),
            ),
            if (_loading) const LinearProgressIndicator(),
            if (_error != null) Text(_error!),
            Expanded(
              child: ListView(
                children: _titles.map((title) => ListTile(title: Text(title))).toList(),
              ),
            ),
          ],
        ),
      );
}
```

### Line-by-line explanation

- `_userId` is editable query input; `_titles` stores returned results.
- `_search` sets loading/error state, then constructs a URI with query map.
- URI construction handles escaping; the request sends GET.
- The JSON list is mapped to title strings.
- `finally` resets the loading flag for both success and failure.
- The `TextField` uses a numeric keyboard; the button disables during the request.
- `Expanded` gives the list a bounded height.
- `dispose` releases the controller.

### Widget tree

```text
Scaffold -> Column
          ├-> TextField(query)
          ├-> Search button
          ├-> optional progress/error
          └-> Expanded ListView
```

### Execution flow

```text
Input -> Uri query encoding -> GET /posts?userId=...
      -> decode list -> extract titles -> render results
```

### ASCII UI preview

```text
+---------------------------+
|     GET with query        |
+---------------------------+
| User ID                   |
| [ 1                    ]  |
| [        Search        ]  |
| • sunt aut facere...      |
+---------------------------+
```

### Example explanation

The query map makes encoding reliable for spaces, ampersands, and non-ASCII characters. The service uses `userId` as its filter name; a real API may use a different contract.

If the URI is built by string concatenation, special characters can corrupt the URL. If the response list is large, use server pagination rather than downloading everything.

**Common mistakes:** Query strings are visible in logs and may be stored by intermediaries; do not put passwords or access tokens there.

**Professional use case:** Search and filtering endpoints with page/sort parameters.

# Complete Example 2

Multiple query parameters:

```dart
Uri.https('api.example.com', '/orders', {
  'status': 'open',
  'page': '3',
  'sort': 'createdAt',
});
```

Values are strings in URI query maps; convert numeric inputs intentionally.

# Complete Example 3

Optional parameters can be omitted:

```dart
final query = <String, String>{if (search.isNotEmpty) 'q': search};
```

Do not send placeholder values such as `"null"` unless the API explicitly defines that meaning.

# Advanced Example

Use cursor pagination where the API supplies opaque cursors. Preserve filters and sort parameters across pages. Encode repeated parameters according to the backend contract; map-based URI construction may not express every repeated-key format.

# Under the Hood

URI query encoding percent-encodes reserved characters. The server parses the query string into values and validates them. Query values are generally logged and are not appropriate for secrets.

# Performance Notes

- Debounce rapidly changing search parameters.
- Use pagination and server filtering.
- Avoid refetching identical queries unnecessarily.
- Cancel or ignore stale search responses.

# Best Practices

- Build URIs with `Uri`.
- Validate parameter ranges and allowed values.
- Keep secrets in headers, not query strings.
- Include query state in cache keys.
- Handle empty result sets.

# Common Mistakes

- Manual URL concatenation.
- Sending `null` as a literal string.
- Ignoring query encoding.
- Forgetting query values in cache keys.
- Applying stale response results after newer searches.

# Debugging

- Print sanitized final URI in development.
- Verify parameter names and encoding.
- Compare request with API contract.
- Check response when filter yields no results.

# Interview Questions

1. What is a query parameter? A key-value option in the URI query section.
2. Why use a URI builder? It safely encodes and composes components.
3. Why not send tokens in query? URLs are frequently logged and exposed.
4. How do you avoid stale search responses? Cancel or sequence requests and apply only the latest result.

# Try It Yourself

1. Add page and page-size parameters.
2. Test search text containing spaces and `&`.
3. Add optional sort.
4. Show an explicit empty result state.

# Challenge Project

Build a filterable product list with category, sort, query, pagination, cache key, and stale-request protection.

# Quick Revision Sheet

- Query parameters express filters/options.
- Use `Uri` for encoding.
- Keep secrets out of URLs.
- Validate ranges and optional values.
- Include all parameters in cache/staleness logic.

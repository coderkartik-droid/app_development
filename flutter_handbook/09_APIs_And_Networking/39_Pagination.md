# Introduction

Pagination divides a large collection into smaller pages. The client requests a subset, displays it, and asks for more as needed.

# Why this concept exists

Downloading an entire large dataset wastes bandwidth, memory, and time. Pagination improves initial load and scales with growing records.

# Real-world analogy

Pagination is reading a book one chapter at a time rather than carrying every page open at once.

# Internal Working

APIs commonly use offset/limit or cursor pagination. Offset identifies a position but can skip/duplicate records as data changes. Cursor pagination uses an opaque token representing the next position and is often more stable for changing feeds.

# Syntax

```dart
Uri.https('api.example.com', '/items', {
  'limit': '$pageSize',
  'cursor': cursor ?? '',
});
```

# Complete Example 1

This runnable app demonstrates offset pagination with a fake repository. A real backend must provide consistent page semantics.

```dart
import 'package:flutter/material.dart';

class PageResult {
  const PageResult(this.items, this.hasMore);
  final List<String> items;
  final bool hasMore;
}

class FakePageRepository {
  Future<PageResult> fetch(int page, int size) async {
    await Future<void>.delayed(const Duration(milliseconds: 300));
    final start = page * size;
    final items = List.generate(size, (index) => 'Item ${start + index + 1}');
    return PageResult(items, page < 2);
  }
}

void main() => runApp(const PaginationApp());

class PaginationApp extends StatelessWidget {
  const PaginationApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: PaginationPage());
}

class PaginationPage extends StatefulWidget {
  const PaginationPage({super.key});

  @override
  State<PaginationPage> createState() => _PaginationPageState();
}

class _PaginationPageState extends State<PaginationPage> {
  final _repository = FakePageRepository();
  final List<String> _items = [];
  int _page = 0;
  bool _loading = false;
  bool _hasMore = true;

  Future<void> _loadNext() async {
    if (_loading || !_hasMore) return;
    setState(() => _loading = true);
    try {
      final result = await _repository.fetch(_page, 10);
      if (!mounted) return;
      setState(() {
        _items.addAll(result.items);
        _page++;
        _hasMore = result.hasMore;
      });
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Pagination')),
        body: Column(
          children: [
            Expanded(
              child: ListView.builder(
                itemCount: _items.length,
                itemBuilder: (context, index) => ListTile(title: Text(_items[index])),
              ),
            ),
            if (_loading) const LinearProgressIndicator(),
            if (_hasMore)
              FilledButton(onPressed: _loadNext, child: const Text('Load next page')),
          ],
        ),
      );
}
```

### Line-by-line explanation

- `PageResult` contains one page plus whether another page exists.
- The fake repository simulates latency and deterministic results.
- `_items` accumulates pages; `_page` tracks offset page; `_loading` prevents duplicate calls.
- `_loadNext` guards concurrent/no-more requests.
- Returned items append, page advances, and `hasMore` updates.
- `ListView.builder` lazily builds rows.
- The button disappears when no more pages remain.

### Widget tree

```text
PaginationPage → Scaffold → Column
                           ├→ Expanded ListView.builder
                           ├→ optional progress
                           └→ Load next button
```

### Execution flow

```text
Load page n -> repository/API -> append records -> increment cursor/page
            -> update hasMore -> show next action or finish
```

### ASCII UI preview

```text
+---------------------------+
|        Pagination         |
+---------------------------+
| Item 1                    |
| Item 2                    |
| ...                       |
| [    Load next page    ]  |
+---------------------------+
```

### Example explanation

The fake page size is ten. It appends items and stops after page two. In production, use server-provided `nextCursor` or `hasMore`, not a hard-coded assumption.

If the loading guard is removed, repeated taps can append duplicate pages. If page state advances before a successful fetch, a failure can skip data.

**Common mistakes:** Offset pagination may shift when new records are inserted while paging.

# Complete Example 2

Cursor response model:

```dart
class CursorPage<T> {
  const CursorPage({required this.items, this.nextCursor});
  final List<T> items;
  final String? nextCursor;
}
```

Null cursor can indicate completion, as defined by the API.

# Complete Example 3

Reset pagination when filters change:

```text
clear current items -> reset cursor -> fetch first page for new filter
```

Do not append results from an old filter to the new collection.

# Advanced Example

Protect against stale and out-of-order requests with query identity or cancellation. Handle duplicate records using stable IDs, and preserve scroll position when refreshing. Define refresh behavior separately from append behavior.

# Under the Hood

The server applies limit and cursor/offset to its query. A cursor may encode ordering state. The client appends results and requests additional pages as needed.

# Performance Notes

- Choose reasonable page sizes.
- Use lazy list widgets.
- Avoid loading pages in parallel unless ordering is guaranteed.
- Deduplicate by stable ID if API can overlap pages.

# Best Practices

- Prefer server-provided cursors for mutable collections.
- Use explicit loading/error state per page operation.
- Reset on filter/sort changes.
- Keep retry on the same cursor after failure.
- Avoid duplicate requests.

# Common Mistakes

- Advancing page before successful result.
- Appending duplicate records.
- Not resetting when filter changes.
- Fetching all pages at startup.
- Assuming final page from item count alone.

# Debugging

- Log page/cursor and item count.
- Check next cursor progression.
- Test empty final page and repeated cursor.
- Simulate failure and retry.

# Interview Questions

1. Offset vs cursor pagination? Position-based vs continuation-token-based.
2. Why prefer cursor for live feeds? More stable when rows are inserted/deleted.
3. How avoid duplicate page requests? Guard loading and track cursor.
4. What when filters change? Clear results and reset cursor.

# Try It Yourself

1. Convert fake repository to cursor pagination.
2. Add a per-page error retry.
3. Deduplicate by stable IDs.
4. Reset paging when sort changes.

# Challenge Project

Build a paginated catalog screen with cursor, filters, refresh, append progress, and robust retry/duplicate handling.

# Quick Revision Sheet

- Pagination bounds response size.
- Use server next cursor/hasMore.
- Guard concurrent loads.
- Reset cursor on query changes.
- Handle retry and duplicate records.

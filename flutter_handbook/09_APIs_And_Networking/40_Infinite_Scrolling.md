# Introduction

Infinite scrolling loads more data as the user approaches the end of a list. It is a UI pattern commonly implemented on top of API pagination.

# Why this concept exists

Users can browse large feeds without tapping a “next page” button. The app fetches only what is needed as the viewport advances.

# Real-world analogy

A conveyor belt brings the next items forward as you reach the end of the current set.

# Internal Working

A `ScrollController` observes scroll position. When the remaining extent falls below a threshold, the app requests the next page if one is available and no request is already active. The API supplies the next cursor/offset. The list appends results and may show a footer spinner or retry button.

# Syntax

```dart
if (controller.position.extentAfter < 400) {
  loadNextPage();
}
```

# Complete Example 1

This runnable sample uses a fake paginated repository and a scroll controller to load more rows automatically.

```dart
import 'package:flutter/material.dart';

class FeedRepository {
  Future<List<String>> load(int page) async {
    await Future<void>.delayed(const Duration(milliseconds: 350));
    return List.generate(15, (index) => 'Page $page · Story ${index + 1}');
  }
}

void main() => runApp(const InfiniteScrollApp());

class InfiniteScrollApp extends StatelessWidget {
  const InfiniteScrollApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: InfiniteScrollPage());
}

class InfiniteScrollPage extends StatefulWidget {
  const InfiniteScrollPage({super.key});

  @override
  State<InfiniteScrollPage> createState() => _InfiniteScrollPageState();
}

class _InfiniteScrollPageState extends State<InfiniteScrollPage> {
  final _scroll = ScrollController();
  final _repository = FeedRepository();
  final List<String> _items = [];
  int _page = 0;
  bool _loading = false;
  bool _hasMore = true;

  @override
  void initState() {
    super.initState();
    _scroll.addListener(_onScroll);
    _loadMore();
  }

  void _onScroll() {
    if (_scroll.hasClients && _scroll.position.extentAfter < 350) {
      _loadMore();
    }
  }

  Future<void> _loadMore() async {
    if (_loading || !_hasMore) return;
    setState(() => _loading = true);
    try {
      final pageItems = await _repository.load(_page);
      if (!mounted) return;
      setState(() {
        _items.addAll(pageItems);
        _page++;
        _hasMore = pageItems.length == 15 && _page < 5;
      });
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  void dispose() {
    _scroll.removeListener(_onScroll);
    _scroll.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Infinite scrolling')),
        body: ListView.builder(
          controller: _scroll,
          itemCount: _items.length + (_hasMore ? 1 : 0),
          itemBuilder: (context, index) {
            if (index == _items.length) {
              return Center(
                child: _loading
                    ? const Padding(
                        padding: EdgeInsets.all(20),
                        child: CircularProgressIndicator(),
                      )
                    : TextButton(
                        onPressed: _loadMore,
                        child: const Text('Load more'),
                      ),
              );
            }
            return ListTile(title: Text(_items[index]));
          },
        ),
      );
}
```

### Line-by-line explanation

- `_scroll` observes the list and is disposed when the page ends.
- `initState` installs one listener and starts the first page.
- `_onScroll` checks `extentAfter`, the remaining content distance.
- `_loadMore` guards duplicate/no-more requests and appends a page.
- The fake repository returns fifteen rows and the demo stops after five pages.
- `ListView.builder` adds a footer slot while more data may exist.
- Footer displays progress or a fallback Load more button.
- Listener removal prevents callbacks after disposal.

### Widget tree

```text
InfiniteScrollPage → Scaffold → ListView.builder
                              ├→ ListTile rows
                              └→ footer progress/retry
```

### Execution flow

```text
Scroll nears end -> extentAfter threshold -> loadMore guard
                  -> fetch cursor/page -> append -> list grows
```

### ASCII UI preview

```text
+---------------------------+
|    Infinite scrolling     |
+---------------------------+
| Page 0 · Story 1          |
| Page 0 · Story 2          |
| ...                       |
|       [spinner]           |
+---------------------------+
```

### Example explanation

The threshold starts loading before the user reaches the exact end, reducing visible waiting. The `_loading` guard prevents a burst of scroll events from launching duplicate requests. The fake page limit is demonstration logic; real APIs supply next-cursor metadata.

If listener cleanup is removed, the controller can retain the state callback. If the guard is removed, several pages may load concurrently and duplicate.

**Common mistakes:** A footer is part of the list item count; ensure its index is never used to access `_items`.

# Complete Example 2

Use a retry footer when the next page fails:

```text
if (appendError != null) show Retry button
else if (loadingMore) show spinner
```

Keep already-loaded content visible after append failure.

# Complete Example 3

For pull-to-refresh, clear/reset cursor and replace results rather than append a new first page to the old end.

# Advanced Example

Handle short initial pages that do not fill the viewport: after a page completes, optionally request another while the user is still near the bottom, with a strict loading and maximum-page guard. Add scroll restoration and deduplicate stable IDs.

# Under the Hood

Scroll notifications update the controller position. `extentAfter` measures content remaining below the viewport. The listener triggers asynchronous data work; appending changes list extent and creates new sliver children lazily.

# Performance Notes

- Use `ListView.builder`.
- Keep row heights/rendering efficient.
- Avoid huge retained lists; consider windowing/eviction.
- Debounce or guard listener threshold.

# Best Practices

- Use server pagination.
- Load before the bottom.
- Guard concurrent requests.
- Expose footer loading/error.
- Remove listeners and dispose controller.
- Preserve scroll and deduplicate results.

# Common Mistakes

- Request on every scroll event without guard.
- Accessing footer index as a data item.
- No end-of-data condition.
- Losing loaded items on append failure.
- Forgetting controller cleanup.

# Debugging

- Log cursor/page and threshold triggers.
- Check footer item count.
- Test tiny screens and short pages.
- Simulate repeated scroll notifications and request failures.

# Interview Questions

1. What does `extentAfter` measure? Content distance remaining after viewport.
2. Why trigger before bottom? Gives network time to fetch before user reaches end.
3. Why guard loading? Prevent duplicate requests.
4. How handle append error? Keep old rows and show retry footer.

# Try It Yourself

1. Add retry footer state.
2. Replace page number with cursor.
3. Add pull-to-refresh.
4. Test pages that are shorter than viewport.

# Challenge Project

Build an infinite news feed with cursor pagination, append loading/error footer, pull-to-refresh, deduplication, and scroll controller lifecycle.

# Quick Revision Sheet

- Infinite scroll is UI over pagination.
- Use extent threshold and load guards.
- Footer belongs in item count.
- Keep prior rows on append errors.
- Dispose controller/listener.

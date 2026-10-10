# Introduction

This project outlines a news reader that loads headlines, displays article summaries, supports refresh/search, and opens article details. News is a paginated, frequently changing API use case.

# Why this concept exists

Feeds combine API requests, model mapping, pagination, caching, and user interaction. A clear structure prevents repeated requests and stale page mixing.

# Real-world analogy

A newsstand replaces the front page as new editions arrive. A reader can browse cached headlines immediately and refresh for the latest edition.

# Internal Working

The repository owns API and cache policy. The screen consumes typed articles and pagination metadata. Search changes query identity and resets cursor. Article content may be fetched separately; do not assume a headline response contains a full article.

# Syntax

```dart
class Article {
  const Article({required this.id, required this.title, required this.summary});
  final String id;
  final String title;
  final String summary;
}
```

# Complete Example 1

This runnable app uses a fake news repository so it runs without external API keys. Replace it with a provider-compliant client and review API terms before production.

```dart
import 'package:flutter/material.dart';

class Article {
  const Article({
    required this.id,
    required this.title,
    required this.summary,
  });
  final String id;
  final String title;
  final String summary;
}

class NewsRepository {
  Future<List<Article>> fetchTopStories() async {
    await Future<void>.delayed(const Duration(milliseconds: 500));
    return const [
      Article(
        id: 'a1',
        title: 'New city garden opens',
        summary: 'Residents gathered for the opening this morning.',
      ),
      Article(
        id: 'a2',
        title: 'Science team shares discovery',
        summary: 'Researchers described their latest findings.',
      ),
    ];
  }
}

void main() => runApp(const NewsApp());

class NewsApp extends StatelessWidget {
  const NewsApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: NewsPage());
}

class NewsPage extends StatefulWidget {
  const NewsPage({super.key});

  @override
  State<NewsPage> createState() => _NewsPageState();
}

class _NewsPageState extends State<NewsPage> {
  final _repository = NewsRepository();
  List<Article> _articles = [];
  bool _loading = false;
  String? _error;

  @override
  void initState() {
    super.initState();
    _load();
  }

  Future<void> _load() async {
    setState(() {
      _loading = true;
      _error = null;
    });
    try {
      final result = await _repository.fetchTopStories();
      if (!mounted) return;
      setState(() => _articles = result);
    } catch (error) {
      if (!mounted) return;
      setState(() => _error = 'Could not load headlines');
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(
          title: const Text('Daily news'),
          actions: [
            IconButton(onPressed: _load, icon: const Icon(Icons.refresh)),
          ],
        ),
        body: _loading && _articles.isEmpty
            ? const Center(child: CircularProgressIndicator())
            : _error != null && _articles.isEmpty
                ? Center(child: Text(_error!))
                : RefreshIndicator(
                    onRefresh: _load,
                    child: ListView(
                      children: [
                        if (_error != null) ListTile(title: Text(_error!)),
                        ..._articles.map(
                          (article) => ListTile(
                            title: Text(article.title),
                            subtitle: Text(article.summary),
                            trailing: const Icon(Icons.chevron_right),
                          ),
                        ),
                      ],
                    ),
                  ),
      );
}
```

### Line-by-line explanation

- `Article` is a typed news summary.
- `NewsRepository` simulates remote latency and returns deterministic headlines.
- The page loads once on startup and stores list/loading/error state.
- `_load` preserves previously loaded articles during a refresh.
- `RefreshIndicator` exposes pull-to-refresh on the scrollable.
- Errors with existing content are shown without immediately discarding the feed.
- The app bar refresh button provides an alternative.

### Widget tree

```text
NewsPage → Scaffold
         ├→ AppBar(refresh)
         └→ loading/error/RefreshIndicator
                            └→ ListView → Article ListTile*
```

### Execution flow

```text
Startup/refresh -> repository -> API/cache -> Article list
                -> preserve old data if refreshing
                -> success replaces feed / failure shows error
```

### ASCII UI preview

```text
+---------------------------+
| Daily news             ↻ |
+---------------------------+
| New city garden opens     |
| Residents gathered...     |
|---------------------------|
| Science team shares...    |
| Researchers described...  |
+---------------------------+
```

### Example explanation

The sample emphasizes refresh behavior and keeps prior articles visible if refreshing fails. It uses a fake repository, not a live news API. Real providers may impose attribution, licensing, rate limits, and restrictions on full-text display.

If previous data is cleared before every refresh, a temporary network error leaves the user with an empty screen. If a full article is assumed from a summary response, navigation may lead to missing content.

**Common mistakes:** Respect copyright and API content-use terms; do not scrape or redistribute content without permission.

# Complete Example 2

Paginate headlines using a server cursor:

```dart
final page = await repository.load(cursor: nextCursor);
```

Append only if the feed query/filter is unchanged.

# Complete Example 3

Open an article using a stable article ID or approved URL. Validate external URLs and use a safe browser/webview policy.

# Advanced Example

Add categories, search debounce, cache timestamps, deduplication, offline banner, saved articles, and account-independent reading preferences. Keep API key and publisher licensing requirements out of public client code where necessary.

# Under the Hood

The news service returns a paginated representation. The repository maps provider DTOs to article models; refresh updates feed state; scroll/pagination requests later pages. Flutter rebuilds visible rows.

# Performance Notes

- Use lazy list construction and image caching.
- Paginate and limit image sizes.
- Keep prior feed during refresh.
- Avoid refetching on every rebuild.

# Best Practices

- Show source, time, and attribution.
- Respect licensing and content terms.
- Handle stale feed and offline reads.
- Use stable IDs and pagination.
- Validate external links.

# Common Mistakes

- Treating demo headlines as real news.
- Ignoring attribution and API terms.
- Duplicating pages after refresh.
- Clearing useful content on transient failure.
- Loading full-resolution images in feed rows.

# Debugging

- Inspect response schema and pagination cursor.
- Check provider quota and attribution rules.
- Test duplicate/removed article IDs.
- Verify refresh failure preserves content.

# Interview Questions

1. Why keep old headlines during refresh? Better resilience during temporary failure.
2. Why use article IDs? Stable navigation and deduplication.
3. What does a news API contract include beyond JSON? Licensing, attribution, quota, and display restrictions.
4. How avoid duplicates across pages? Deduplicate by stable ID.

# Try It Yourself

1. Add category filters.
2. Add a timestamp and source.
3. Add saved article IDs.
4. Simulate failure while preserving current feed.

# Challenge Project

Build a paginated news reader with source attribution, offline cache, refresh, search, dedupe, saved stories, and compliant external article links.

# Quick Revision Sheet

- Use typed article DTOs/models and repository.
- Refresh should preserve usable data.
- Paginate and deduplicate.
- Respect provider terms and attribution.
- Cache with freshness metadata.

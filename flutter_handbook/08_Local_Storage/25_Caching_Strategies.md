# Introduction

A cache is a temporary or refreshable copy of data kept closer to the app so it can be read faster or used offline. Unlike authoritative user data, cached data is usually safe to recreate from another source.

# Why this concept exists

Network requests add latency, consume bandwidth, and can fail. A cache improves perceived speed and availability, provided the app has a clear rule for when cached values are fresh enough.

# Real-world analogy

A chef keeps commonly used ingredients on the counter. The counter cache is fast to reach, but ingredients expire and must be replenished from the pantry.

# Internal Working

A cache needs a key, value, freshness metadata, and an invalidation or refresh policy. Common patterns include cache-first, network-first, stale-while-revalidate, and write-through.

Local memory caches disappear with process death. Disk caches can survive restarts. A cache should not become an accidental source of truth when remote data has changed.

# Syntax

```dart
class CacheEntry<T> {
  const CacheEntry({required this.value, required this.savedAt});
  final T value;
  final DateTime savedAt;
}
```

# Complete Example 1

This runnable UI demonstrates stale-while-revalidate with a small in-memory cache interface. The fake repository simulates latency; replace it with a persistent cache plus a real API for production.

```dart
import 'package:flutter/material.dart';

class Article {
  const Article(this.title, this.summary);
  final String title;
  final String summary;
}

class ArticleRepository {
  Article? _cache;

  Future<Article?> readCache() async => _cache;

  Future<Article> fetchRemote() async {
    await Future<void>.delayed(const Duration(seconds: 1));
    const fresh = Article('Fresh article', 'Loaded from the simulated network.');
    _cache = fresh;
    return fresh;
  }
}

void main() => runApp(const CacheApp());

class CacheApp extends StatelessWidget {
  const CacheApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: CachePage());
}

class CachePage extends StatefulWidget {
  const CachePage({super.key});

  @override
  State<CachePage> createState() => _CachePageState();
}

class _CachePageState extends State<CachePage> {
  final _repository = ArticleRepository();
  Article? _article;
  bool _refreshing = false;
  String? _error;

  @override
  void initState() {
    super.initState();
    _refresh();
  }

  Future<void> _refresh() async {
    final cached = await _repository.readCache();
    if (mounted && cached != null) setState(() => _article = cached);
    if (mounted) setState(() {
      _refreshing = true;
      _error = null;
    });
    try {
      final fresh = await _repository.fetchRemote();
      if (!mounted) return;
      setState(() {
        _article = fresh;
        _refreshing = false;
      });
    } catch (error) {
      if (!mounted) return;
      setState(() {
        _refreshing = false;
        _error = 'Refresh failed; showing saved data if available.';
      });
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Cache strategy')),
        body: Center(
          child: _article == null && _refreshing
              ? const CircularProgressIndicator()
              : Column(
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    if (_article != null) ...[
                      Text(_article!.title),
                      Text(_article!.summary),
                    ],
                    if (_refreshing) const LinearProgressIndicator(),
                    if (_error != null) Text(_error!),
                    FilledButton(
                      onPressed: _refreshing ? null : _refresh,
                      child: const Text('Refresh'),
                    ),
                  ],
                ),
        ),
      );
}
```

### Line-by-line explanation

- `Article` is the small data model displayed by the page.
- `ArticleRepository` owns a fake in-memory cache and remote simulation.
- `readCache` returns the cached object immediately.
- `fetchRemote` simulates network delay, creates fresh data, and replaces the cache.
- The page loads through `_refresh` from `initState`.
- `_refresh` displays cached content first when present, marks refreshing, then fetches fresh data.
- Errors retain the current article and show a refresh message.
- The UI distinguishes initial loading, cached/fresh content, refresh progress, and error.
- The refresh button is disabled while a request is in flight.

### Widget tree

```text
MaterialApp → CachePage → Scaffold → Center
                                  ├→ progress indicator
                                  └→ Column(content, refresh, error, button)
```

### Execution flow

```text
Read cache → render immediately
      |
      v
Fetch remote → success replaces cache/content
             └→ failure keeps cached content + error
```

### ASCII phone preview

```text
+---------------------------+
|       Cache strategy     |
+---------------------------+
| Fresh article             |
| Loaded from network.      |
| [======= refreshing]      |
| [        Refresh       ]  |
+---------------------------+
```

### Example explanation

Stale-while-revalidate prioritizes quick display, then refreshes in the background. This demo's cache is memory-only and disappears at process death; a persistent cache would store the entry and timestamp in a database/file.

If stale content is removed before fetching, offline startup has no immediate content. If refresh errors are swallowed, the user cannot tell whether the data may be outdated. Avoid using this policy for data where stale values are unsafe.

**Common mistakes:** Caching without expiration or invalidation creates indefinitely stale data.

**Professional use case:** Article feeds, product catalogs, and profile data where old content is useful while fresh content loads.

# Complete Example 2

Cache-first behavior:

```text
Read local value
  ├─ fresh → return it
  └─ stale/missing → fetch remote, store, return
```

This saves network calls but must have a freshness definition and manual refresh strategy.

# Complete Example 3

Network-first behavior:

```text
Try remote request
  ├─ success → store and return fresh result
  └─ failure → return usable cached result or explicit error
```

Use for content where freshness is more important but offline fallback remains helpful.

# Advanced Example

Choose cache keys that include all request dimensions (account, query, locale, filters). Set TTL or validators (ETag/Last-Modified) as appropriate. Evict by size or age, and avoid showing one user's cached response to another account.

# Under the Hood

The repository mediates memory/disk/network sources. Freshness metadata determines whether a cache hit is acceptable. Revalidation can reduce transfer size; conflict and invalidation rules are application policy.

# Performance Notes

- Cache only high-value data.
- Bound cache size and define eviction.
- Avoid parsing large payloads on the UI isolate.
- Reuse HTTP validators where supported.
- Measure hit rate and refresh latency.

# Best Practices

- Classify data as authoritative, cache, or user-authored.
- Define freshness, invalidation, and error fallback.
- Scope cache entries by account and query.
- Keep cached display labeled when staleness matters.
- Provide manual refresh for relevant content.

# Common Mistakes

- Treating cache as permanent truth.
- No expiration or eviction policy.
- Cache key omits account/filter/locale.
- Replacing useful data with empty response after a temporary failure.
- Logging cached private content.

# Debugging

- Inspect cache key, saved time, and refresh result.
- Test network unavailable, slow, and error paths.
- Switch accounts and check cache isolation.
- Measure cache hits and stale fallbacks.

# Interview Questions

1. What does stale-while-revalidate do? Shows cached data while refreshing it in the background.
2. What is cache invalidation? Determining when an entry is no longer acceptable.
3. Why include user/account in a key? To prevent cross-account data leakage.
4. How can HTTP validators help? They can allow conditional refresh with reduced data transfer.

# Try It Yourself

1. Add a `savedAt` timestamp and display stale status.
2. Add a cache TTL.
3. Test offline startup and failed refresh.
4. Define an eviction policy for a bounded cache.

# Challenge Project

Create a repository using stale-while-revalidate for a catalog, with persistent cache, account-scoped keys, timestamps, manual refresh, and clear behavior on sign-out.

# Quick Revision Sheet

- Cache is a refreshable copy, not source of truth.
- Choose cache-first, network-first, or stale-while-revalidate.
- Define key, freshness, invalidation, fallback, and eviction.
- Scope by account and request parameters.
- Test offline, stale, and failed refresh.

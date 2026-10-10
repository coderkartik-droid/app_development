# Introduction

API caching keeps a previously fetched response so the app can reuse it rather than always waiting for the network. A cache may live in memory or on disk.

# Why this concept exists

Caching reduces latency, network usage, and repeated work. It can also show useful content when a request is temporarily unavailable.

# Real-world analogy

A café keeps a copy of today's menu near the counter. It saves time, but the menu needs a date because prices or availability can change.

# Internal Working

A cache entry needs a key, payload, saved time, and freshness policy. HTTP headers such as Cache-Control, ETag, and Last-Modified can guide reuse and validation. Application caches must account for user identity and query parameters.

# Syntax

```dart
if (entry != null && entry.isFresh) return entry.value;
final fresh = await api.fetch();
await cache.write(key, fresh);
```

# Complete Example 1

This runnable sample demonstrates an in-memory cache with a one-minute time-to-live. It uses a fake API; replace it with HTTP and persistent storage in production.

```dart
import 'package:flutter/material.dart';

class CacheEntry<T> {
  const CacheEntry(this.value, this.savedAt);
  final T value;
  final DateTime savedAt;

  bool get isFresh =>
      DateTime.now().difference(savedAt) < const Duration(minutes: 1);
}

class NameRepository {
  CacheEntry<String>? _cache;

  Future<String> loadName() async {
    final entry = _cache;
    if (entry != null && entry.isFresh) return entry.value;
    await Future<void>.delayed(const Duration(milliseconds: 500));
    const name = 'Avery';
    _cache = CacheEntry(name, DateTime.now());
    return name;
  }
}

void main() => runApp(const ApiCacheApp());

class ApiCacheApp extends StatelessWidget {
  const ApiCacheApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: ApiCachePage());
}

class ApiCachePage extends StatefulWidget {
  const ApiCachePage({super.key});

  @override
  State<ApiCachePage> createState() => _ApiCachePageState();
}

class _ApiCachePageState extends State<ApiCachePage> {
  final _repository = NameRepository();
  String _message = 'Not loaded';

  Future<void> _load() async {
    setState(() => _message = 'Loading...');
    final value = await _repository.loadName();
    if (mounted) setState(() => _message = 'Name: $value');
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('API cache')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_message),
              FilledButton(onPressed: _load, child: const Text('Load')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `CacheEntry` stores payload and the time it was saved.
- `isFresh` compares the current time with a one-minute TTL.
- `NameRepository.loadName` returns a fresh cached value or simulates a fetch and replaces the cache.
- The page calls the repository and displays loading/result.
- The cache is process-memory only and disappears after app restart.

### Widget tree

```text
MaterialApp → ApiCachePage → Scaffold → Center → Column(status, button)
```

### Execution flow

```text
Load -> cache exists and fresh? -> return cached
                               \-> fetch -> store timestamped value -> return
```

### ASCII UI preview

```text
+---------------------------+
|          API cache        |
+---------------------------+
| Name: Avery               |
| [          Load         ] |
+---------------------------+
```

### Example explanation

TTL bounds the time a value can be considered fresh. A production cache should also handle invalidation after mutations, account scoping, and network failure fallback.

If the timestamp is omitted, freshness cannot be determined. If the key does not include request parameters or account, unrelated responses can be mixed.

**Common mistakes:** Do not cache sensitive data under a shared key across accounts.

# Complete Example 2

Use ETag revalidation:

```text
GET with If-None-Match: saved-etag
304 Not Modified -> reuse cached body
200 OK -> replace cache and ETag
```

This saves transfer size while checking freshness.

# Complete Example 3

Invalidate after successful mutation:

```dart
await api.updateProfile(profile);
await cache.remove('profile:${profile.id}');
```

Or update the cached representation from the confirmed response.

# Advanced Example

Choose cache-first, network-first, or stale-while-revalidate per resource. Define maximum size, eviction, privacy scope, and cache consistency. Avoid serving stale account data during sign-out or account switching.

# Under the Hood

HTTP caches use response directives and validators. Application caches use their own keys and TTLs. Revalidation can confirm a stored representation without transferring the entire body.

# Performance Notes

- Measure hit rate and stale fallback usefulness.
- Bound memory and disk cache.
- Avoid serializing large objects repeatedly.
- Deduplicate in-flight requests.

# Best Practices

- Key by all request dimensions and account.
- Define freshness and invalidation.
- Respect server cache headers.
- Label stale data when relevant.
- Clear account-scoped cache on sign-out.

# Common Mistakes

- No expiration/eviction.
- Cache key omits filters or user.
- Serving stale data as current.
- Forgetting invalidation after writes.
- Caching errors as valid content.

# Debugging

- Inspect cache key, saved time, validator, hit/miss.
- Test mutation invalidation.
- Switch accounts and verify isolation.
- Simulate stale response and offline fallback.

# Interview Questions

1. What metadata belongs in a cache entry? Key, value, saved time/freshness metadata.
2. What is ETag? A representation validator used for conditional requests.
3. Why invalidate after mutation? Cached data may no longer match server state.
4. What should cache key include? All response-affecting parameters and account scope.

# Try It Yourself

1. Add a refresh action that bypasses cache.
2. Add a TTL and stale label.
3. Clear cache after profile update.
4. Add account ID to cache key.

# Challenge Project

Build a repository cache with TTL, ETag revalidation, mutation invalidation, account isolation, and offline fallback tests.

# Quick Revision Sheet

- Cache improves speed/offline resilience.
- Define key, TTL, invalidation, eviction.
- Use ETag/Last-Modified when available.
- Include account and query dimensions.
- Never confuse stale cache with authority.

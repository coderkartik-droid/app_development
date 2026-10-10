# Introduction

The repository pattern presents a clean interface for loading and changing data without exposing whether it came from a network, database, cache, or combination.

# Why this concept exists

Screens should focus on user interaction and presentation. Repositories make data-source changes, testing, caching, and offline behavior easier to manage.

# Real-world analogy

A librarian retrieves a book for a reader without requiring the reader to know which shelf, archive, or lending system supplied it.

# Internal Working

A repository accepts app-level parameters, calls one or more data sources, maps results, and applies policies such as cache-first or remote refresh. It returns typed values or errors to state management.

# Syntax

```dart
abstract interface class WeatherRepository {
  Future<Weather> getCurrent(String city);
}
```

# Complete Example 1

This runnable app uses a repository contract with a fake implementation. Replace the fake with an HTTP-backed repository for production.

```dart
import 'package:flutter/material.dart';

class Weather {
  const Weather({required this.city, required this.temperature});
  final String city;
  final int temperature;
}

abstract interface class WeatherRepository {
  Future<Weather> getCurrent(String city);
}

class FakeWeatherRepository implements WeatherRepository {
  @override
  Future<Weather> getCurrent(String city) async {
    await Future<void>.delayed(const Duration(milliseconds: 400));
    return Weather(city: city, temperature: 18);
  }
}

void main() => runApp(const RepositoryApp());

class RepositoryApp extends StatelessWidget {
  const RepositoryApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        home: WeatherPage(repository: FakeWeatherRepository()),
      );
}

class WeatherPage extends StatefulWidget {
  const WeatherPage({required this.repository, super.key});
  final WeatherRepository repository;

  @override
  State<WeatherPage> createState() => _WeatherPageState();
}

class _WeatherPageState extends State<WeatherPage> {
  Weather? _weather;
  bool _loading = false;

  Future<void> _load() async {
    setState(() => _loading = true);
    try {
      final result = await widget.repository.getCurrent('Wellington');
      if (mounted) setState(() => _weather = result);
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Repository pattern')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              if (_loading) const CircularProgressIndicator(),
              if (_weather != null)
                Text('${_weather!.city}: ${_weather!.temperature}°'),
              FilledButton(onPressed: _load, child: const Text('Load weather')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `Weather` is a typed app model.
- `WeatherRepository` is the contract.
- `FakeWeatherRepository` simulates latency and deterministic data.
- The app injects the repository into `WeatherPage`.
- `_load` requests a city and stores the result if the page remains mounted.
- `finally` always resets loading.
- The UI renders loading, result, and action.

### Widget tree

```text
RepositoryApp → WeatherPage(repository)
              → Scaffold → Column(progress, weather, button)
```

### Execution flow

```text
Page event -> repository interface -> fake/network/cache implementation
           <- typed Weather <- render
```

### ASCII UI preview

```text
+---------------------------+
|   Repository pattern      |
+---------------------------+
| Wellington: 18°           |
| [     Load weather      ] |
+---------------------------+
```

### Example explanation

The screen depends on behavior, not a network package. A test fake can replace the repository. Production implementation can combine API and cache without rewriting UI.

If the fake is mistaken for a real API, data is not persistent or live. If the interface is omitted, screen tests may require network access.

**Common mistakes:** Repositories should expose useful domain operations, not simply mirror every low-level HTTP method.

# Complete Example 2

A repository can prefer cache:

```dart
final cached = await cache.read(key);
if (cached != null && !cached.isStale) return cached.value;
final fresh = await api.fetch(key);
await cache.write(key, fresh);
return fresh;
```

The cache freshness and failure fallback policy must be explicit.

# Complete Example 3

Map wire DTO to domain model:

```dart
final dto = await api.getWeather(city);
return Weather.fromDto(dto);
```

This is useful when API structure differs from the app's domain.

# Advanced Example

Use separate local and remote data sources behind the repository for offline-first features. Define conflict, retry, and partial failure policies. Avoid hiding important distinctions—such as stale versus fresh data—if the UI needs to communicate them.

# Under the Hood

The repository coordinates source selection and conversion. It may emit one-shot futures or reactive streams. It should not own widget lifecycle, and its dependencies should have explicit lifetimes.

# Performance Notes

- Cache at appropriate boundaries.
- Deduplicate in-flight requests.
- Use pagination.
- Avoid caching invalid or account-mismatched data.

# Best Practices

- Depend on interfaces where useful.
- Inject implementations.
- Keep domain operations clear.
- Map storage/HTTP errors once.
- Test cache, network, and fallback paths.

# Common Mistakes

- Repository containing presentation logic.
- A pass-through class with no value.
- Hidden stale-data behavior.
- No dependency injection.
- Mixing different account caches.

# Debugging

- Trace which data source served a value.
- Log cache hit/miss and request IDs, not private content.
- Test offline and remote failure paths.
- Inspect stale and invalidation policy.

# Interview Questions

1. What problem does repository solve? Abstracts data sources from UI/domain consumers.
2. Can it combine local and remote sources? Yes.
3. Should it contain widget code? No.
4. How test it? Inject fake API/cache sources and test policies.

# Try It Yourself

1. Add a cache-first method.
2. Add a fake that throws.
3. Test remote failure with stale cache.
4. Define account scope for cache keys.

# Challenge Project

Implement a repository that returns cached data immediately and refreshes from a fake API, exposing freshness and errors to its caller.

# Quick Revision Sheet

- Repository abstracts data sources.
- UI consumes domain-oriented methods.
- Inject fakes for tests.
- Define cache and fallback policy.
- Keep lifecycle and presentation outside repository.

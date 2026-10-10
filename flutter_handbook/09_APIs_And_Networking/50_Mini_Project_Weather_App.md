# Introduction

This project builds a weather lookup interface. The user enters a city, the app requests weather data, maps the response into a model, and displays temperature and description.

# Why this concept exists

Weather is a familiar example of remote data that changes over time, depends on query parameters, and benefits from clear loading and error states.

# Real-world analogy

The app asks a weather station for the forecast for a named city, then presents the measurements in a readable way.

# Internal Working

The UI validates the city, a repository constructs the request, the server authenticates/validates and returns JSON, and the client maps it to a typed model. Real weather services usually require an API key; never embed a privileged secret without understanding that mobile binaries can be inspected. Prefer a backend proxy or provider-approved restricted client key.

# Syntax

```dart
Uri.https('api.example.com', '/weather', {'city': city});
```

# Complete Example 1

This complete app uses a deterministic fake repository so it runs without an API key. Replace the repository with a real weather provider adapter.

```dart
import 'package:flutter/material.dart';

class Weather {
  const Weather({
    required this.city,
    required this.temperatureC,
    required this.description,
  });
  final String city;
  final double temperatureC;
  final String description;
}

class WeatherRepository {
  Future<Weather> fetch(String city) async {
    await Future<void>.delayed(const Duration(milliseconds: 500));
    if (city.toLowerCase() == 'unknown') {
      throw StateError('City was not found');
    }
    return Weather(
      city: city,
      temperatureC: 18.5,
      description: 'Partly cloudy',
    );
  }
}

void main() => runApp(const WeatherApp());

class WeatherApp extends StatelessWidget {
  const WeatherApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: WeatherPage());
}

class WeatherPage extends StatefulWidget {
  const WeatherPage({super.key});

  @override
  State<WeatherPage> createState() => _WeatherPageState();
}

class _WeatherPageState extends State<WeatherPage> {
  final _city = TextEditingController(text: 'Wellington');
  final _repository = WeatherRepository();
  Weather? _weather;
  bool _loading = false;
  String? _error;

  Future<void> _search() async {
    final city = _city.text.trim();
    if (city.isEmpty) {
      setState(() => _error = 'Enter a city name');
      return;
    }
    setState(() {
      _loading = true;
      _error = null;
    });
    try {
      final result = await _repository.fetch(city);
      if (!mounted) return;
      setState(() => _weather = result);
    } catch (error) {
      if (!mounted) return;
      setState(() => _error = 'Weather is unavailable for that city');
    } finally {
      if (mounted) setState(() => _loading = false);
    }
  }

  @override
  void dispose() {
    _city.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Weather lookup')),
        body: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            children: [
              TextField(
                controller: _city,
                decoration: const InputDecoration(labelText: 'City'),
                onSubmitted: (_) => _search(),
              ),
              FilledButton(
                onPressed: _loading ? null : _search,
                child: const Text('Get weather'),
              ),
              if (_loading) const CircularProgressIndicator(),
              if (_error != null) Text(_error!),
              if (_weather != null && !_loading) ...[
                Text(_weather!.city, style: Theme.of(context).textTheme.headlineMedium),
                Text('${_weather!.temperatureC.toStringAsFixed(1)} °C'),
                Text(_weather!.description),
              ],
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `Weather` models city, Celsius temperature, and description.
- `WeatherRepository` is a fake asynchronous source with a deterministic not-found case.
- The page owns city input, data, loading, and error state.
- `_search` trims/validates input and clears old error.
- The repository result is applied only if the page remains mounted.
- `finally` resets loading.
- The UI handles field, action, loading, error, and weather display.
- The text controller is disposed.

### Widget tree

```text
WeatherPage → Scaffold → Padding → Column
                                  ├→ city TextField
                                  ├→ Get weather button
                                  ├→ loading/error
                                  └→ Weather result
```

### Execution flow

```text
City input -> validate -> repository/API -> response DTO -> Weather model
           -> loading/error/result state -> widgets
```

### ASCII UI preview

```text
+---------------------------+
|       Weather lookup      |
+---------------------------+
| City                      |
| [ Wellington           ]  |
| [     Get weather      ]  |
| Wellington                |
| 18.5 °C                   |
| Partly cloudy              |
+---------------------------+
```

### Example explanation

The fake repository keeps the app runnable and avoids a real API key. For production, map provider units explicitly, handle geocoding ambiguity, and show when data was last updated.

If the repository is replaced with hard-coded test output and not disclosed, users may assume live forecasts. If errors are hidden, invalid city names look like empty weather.

**Common mistakes:** Temperature units must be explicit, and location permission is not needed for a city text field.

# Complete Example 2

Map provider response:

```dart
final celsius = (json['current']['temp_c'] as num).toDouble();
```

Validate nested maps and check the provider's unit/field contract.

# Complete Example 3

Use coordinates instead of city text for precise local forecasts:

```text
GET /forecast?lat=...&lon=...
```

Request location permission only with clear user value and consent.

# Advanced Example

Add provider attribution, cache duration, rate-limit handling, stale/offline display, localization, unit preferences, geocoding disambiguation, and privacy-aware location handling. Keep provider API secrets behind a backend when required.

# Under the Hood

Weather requests use HTTPS and query parameters. JSON is decoded to DTOs; unit conversion and domain mapping produce the presentation model. Flutter renders state after asynchronous completion.

# Performance Notes

- Cache by normalized city/coordinates and units.
- Debounce autocomplete, not necessarily explicit submit.
- Avoid unnecessary location polling.
- Respect provider quotas.

# Best Practices

- Use typed model and repository.
- Show units and update time.
- Handle ambiguous city and no data.
- Protect API credentials and obey provider terms.
- Request location access only when necessary.

# Common Mistakes

- Exposing unrestricted API keys.
- Assuming one city name is unique.
- Mixing Fahrenheit/Celsius.
- Ignoring attribution, quota, and stale data.

# Debugging

- Inspect sanitized query and status.
- Verify key configuration without printing key.
- Check units and timestamp parsing.
- Test unknown city, timeout, and rate limit.

# Interview Questions

1. Why use a fake repository in the sample? Runnable UI without external credentials.
2. How handle weather units? Map them explicitly and label them.
3. How protect API keys? Avoid embedding privileged keys; proxy or restrict according to provider.
4. Why cache weather? Reduce latency and quota use, with a freshness timestamp.

# Try It Yourself

1. Add Fahrenheit preference.
2. Add last-updated time.
3. Add retry after network error.
4. Compare city and coordinate lookup.

# Challenge Project

Build a weather dashboard with current conditions and forecast, explicit units, caching, loading/error states, and a secure provider integration plan.

# Quick Revision Sheet

- Weather UI: query -> model -> state -> display.
- Units, freshness, attribution, and quotas matter.
- Use repository and cache.
- Protect provider credentials.
- Location access should be optional and justified.

# Project Overview

Build a weather dashboard that displays a location, current temperature, and a short forecast. To keep the code runnable without API keys, this chapter uses clearly labeled sample data. The UI is structured so a real weather data source can be added later.

# Features

- Current conditions and location
- High/low sample values
- Three-day forecast list
- Refresh action
- Loading and error states

# Folder Structure

```text
lib/
  main.dart
  models/weather.dart
  data/weather_repository.dart
  screens/weather_page.dart
  widgets/forecast_tile.dart
```

# ASCII UI Preview

```text
+--------------------------+
| Weather           [↻]    |
| Example City             |
|          ☀               |
|          22°              |
|      Clear skies         |
| H:25°  L:16°             |
|--------------------------|
| Today        ☀   22°     |
| Tomorrow     ☁   19°     |
| Friday       🌧   17°     |
+--------------------------+
```

# Step-by-Step Development Plan

1. Define typed current-weather and forecast models.
2. Keep sample data in a repository, not inside each widget.
3. Model loading, success, and error states explicitly.
4. Render a readable dashboard and forecast list.
5. Replace the sample repository with a real provider and secure configuration.

# Complete Runnable Code

Paste this into `lib/main.dart`. No external package or API key is needed; displayed weather is illustrative, not live data.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const WeatherApp());

class WeatherApp extends StatelessWidget {
  const WeatherApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        title: 'Weather',
        theme: ThemeData(colorSchemeSeed: Colors.blue, useMaterial3: true),
        home: const WeatherPage(),
      );
}

class ForecastDay {
  const ForecastDay({
    required this.day,
    required this.icon,
    required this.summary,
    required this.temperature,
  });
  final String day;
  final IconData icon;
  final String summary;
  final int temperature;
}

class WeatherPage extends StatefulWidget {
  const WeatherPage({super.key});

  @override
  State<WeatherPage> createState() => _WeatherPageState();
}

class _WeatherPageState extends State<WeatherPage> {
  bool _loading = false;
  String? _error;
  int _refreshCount = 0;
  final List<ForecastDay> _forecast = const [
    ForecastDay(day: 'Today', icon: Icons.wb_sunny, summary: 'Clear', temperature: 22),
    ForecastDay(day: 'Tomorrow', icon: Icons.cloud, summary: 'Cloudy', temperature: 19),
    ForecastDay(day: 'Friday', icon: Icons.umbrella, summary: 'Light rain', temperature: 17),
  ];

  Future<void> _refresh() async {
    setState(() {
      _loading = true;
      _error = null;
    });
    await Future<void>.delayed(const Duration(milliseconds: 500));
    if (!mounted) return;
    setState(() {
      _loading = false;
      _refreshCount++;
    });
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(
          title: const Text('Weather'),
          actions: [
            IconButton(
              tooltip: 'Refresh sample forecast',
              onPressed: _loading ? null : _refresh,
              icon: const Icon(Icons.refresh),
            ),
          ],
        ),
        body: ListView(
          padding: const EdgeInsets.all(20),
          children: [
            Text('Example City', style: Theme.of(context).textTheme.titleLarge),
            const SizedBox(height: 24),
            const Icon(Icons.wb_sunny, size: 72, color: Colors.orange),
            Text('22°', textAlign: TextAlign.center,
                style: Theme.of(context).textTheme.displayLarge),
            const Text('Clear skies · H: 25°  L: 16°', textAlign: TextAlign.center),
            const SizedBox(height: 20),
            if (_loading) const LinearProgressIndicator(),
            if (_error != null) Text(_error!),
            if (!_loading && _error == null)
              Text('Sample data · refreshed $_refreshCount time(s)'),
            const Divider(height: 32),
            for (final item in _forecast)
              ListTile(
                leading: Icon(item.icon),
                title: Text(item.day),
                subtitle: Text(item.summary),
                trailing: Text('${item.temperature}°'),
              ),
          ],
        ),
      );
}
```

# Line-by-Line Explanation

- `ForecastDay` is an immutable typed model for one forecast entry.
- `_loading`, `_error`, and `_refreshCount` describe the screen's asynchronous state.
- `_forecast` is illustrative local sample data; it is not fetched from a service.
- `_refresh` marks the screen loading, waits briefly to demonstrate an async lifecycle, then checks `mounted` before updating state.
- The refresh action is disabled during the pending operation to prevent duplicate taps.
- `ListView` keeps the dashboard scrollable.
- The current conditions and forecast rows are built from data fields rather than repeated hard-coded widgets.

The delay is only a UI demonstration, not a weather request. Removing the `mounted` guard can cause a state update after the page is removed. Removing loading feedback makes refresh appear unresponsive.

# Widget Tree

```text
WeatherApp > MaterialApp > WeatherPage > Scaffold
├── AppBar > Refresh IconButton
└── ListView
    ├── Location and current conditions
    ├── Loading/error/sample status
    └── Forecast ListTiles
```

# State Management Used

Local `StatefulWidget` state handles a single screen. A real data source should be represented by a repository and a state model with loading, success, empty, and failure states.

# API/Database Used (if applicable)

No API or database is used. This avoids embedding credentials and keeps the sample runnable. For a real project, use a weather provider's documented client, validate units/time zones, and keep privileged secrets out of the app bundle.

# Common Mistakes

- Presenting sample values as live weather.
- Hard-coding a private API key in source code.
- Treating a timeout or bad response as valid empty weather.
- Forgetting units, location permissions, or stale-data indicators.

# Improvements

- Add location selection and unit conversion.
- Add a real repository with request cancellation and error mapping.
- Cache the last successful forecast in a local database.
- Include accessibility labels and severe-weather states.

# Challenge Version

Implement a real provider behind a repository, cache successful responses in SQLite, and render stale cached data with a clear “updated at” timestamp when offline.

# Summary

This project practices typed data models and asynchronous UI states without requiring a live service. Its sample-data label is essential: placeholder data must never be mistaken for a forecast.

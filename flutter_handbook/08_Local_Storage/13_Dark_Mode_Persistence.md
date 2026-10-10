# Introduction

Dark-mode persistence remembers the user's light, dark, or system theme choice between launches. It is a useful example of combining a preference with app-level state.

# Why this concept exists

Users expect a chosen appearance to remain stable. Resetting to light mode after every launch makes the app feel unreliable and ignores accessibility preferences.

# Real-world analogy

It is like remembering a room's lighting preference. The choice should apply throughout the house, not only on the switch page.

# Internal Working

Persist an enum-like value such as `system`, `light`, or `dark`. On startup, read and validate it, then supply the corresponding `ThemeMode` to `MaterialApp`. When the user changes the option, write it and update observable app state.

Using `ThemeMode.system` lets Flutter follow the platform brightness. A saved choice of “system” is different from a missing preference only if the product distinguishes them.

# Syntax

```dart
ThemeMode get mode => switch (storedName) {
  'light' => ThemeMode.light,
  'dark' => ThemeMode.dark,
  _ => ThemeMode.system,
};
```

# Complete Example 1

This app offers system, light, and dark choices and stores the selected mode. Add `shared_preferences`.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const ThemePreferenceApp());

class ThemePreferenceApp extends StatefulWidget {
  const ThemePreferenceApp({super.key});

  @override
  State<ThemePreferenceApp> createState() => _ThemePreferenceAppState();
}

class _ThemePreferenceAppState extends State<ThemePreferenceApp> {
  ThemeMode _mode = ThemeMode.system;
  bool _loaded = false;

  @override
  void initState() {
    super.initState();
    _loadMode();
  }

  Future<void> _loadMode() async {
    final prefs = await SharedPreferences.getInstance();
    final name = prefs.getString('themeMode') ?? 'system';
    final mode = switch (name) {
      'light' => ThemeMode.light,
      'dark' => ThemeMode.dark,
      _ => ThemeMode.system,
    };
    if (!mounted) return;
    setState(() {
      _mode = mode;
      _loaded = true;
    });
  }

  Future<void> _saveMode(ThemeMode mode) async {
    final name = switch (mode) {
      ThemeMode.light => 'light',
      ThemeMode.dark => 'dark',
      ThemeMode.system => 'system',
    };
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString('themeMode', name);
    if (!mounted) return;
    setState(() => _mode = mode);
  }

  @override
  Widget build(BuildContext context) => MaterialApp(
        theme: ThemeData(colorSchemeSeed: Colors.indigo, brightness: Brightness.light),
        darkTheme: ThemeData(colorSchemeSeed: Colors.indigo, brightness: Brightness.dark),
        themeMode: _mode,
        home: Scaffold(
          appBar: AppBar(title: const Text('Theme preference')),
          body: !_loaded
              ? const Center(child: CircularProgressIndicator())
              : Column(
                  children: ThemeMode.values.map((mode) {
                    return RadioListTile<ThemeMode>(
                      title: Text(mode.name),
                      value: mode,
                      groupValue: _mode,
                      onChanged: (value) {
                        if (value != null) _saveMode(value);
                      },
                    );
                  }).toList(),
                ),
        ),
      );
}
```

### Line-by-line explanation

- `_mode` stores the current app theme policy; `_loaded` prevents showing the selector before restoration.
- `_loadMode` reads a string and maps unknown values safely to system mode.
- `_saveMode` maps the enum to a stable string representation, awaits persistence, then updates app state.
- `MaterialApp` defines both light and dark themes and consumes `themeMode`.
- `RadioListTile` presents mutually exclusive choices with labels.

### Widget tree

```text
ThemePreferenceApp
└── MaterialApp(theme, darkTheme, themeMode)
    └── Scaffold
        └── Column
            └── RadioListTile* (system/light/dark)
```

### Execution flow

```text
Startup → read string → map/validate → MaterialApp theme
Choose mode → encode enum → persist → update root state
```

### ASCII phone preview

```text
+---------------------------+
|    Theme preference       |
+---------------------------+
| ( ) system                |
| ( ) light                 |
| (*) dark                  |
+---------------------------+
```

### Example explanation

The saved representation is a stable string rather than an enum index. If enum ordering changes, stored indexes can silently map to a different value. Unknown strings fall back to system mode.

If `themeMode` is omitted from `MaterialApp`, changing state does not switch app appearance. If startup loading is omitted, the app can flash the wrong theme before the preference is restored.

**Common mistakes:** Theme colors must be designed for both brightness modes; changing `ThemeMode` cannot fix hard-coded low-contrast colors.

**Professional use case:** Global appearance settings for apps with a theme selector.

# Complete Example 2

Use `ThemeMode.system` as the default:

```dart
final stored = prefs.getString('themeMode');
final mode = stored == null ? ThemeMode.system : decodeTheme(stored);
```

This follows device appearance unless the user explicitly selected another mode.

# Complete Example 3

When using an inherited settings controller, widgets can read the mode with a listenable provider. The app should still have a single owner responsible for persistence and validation.

# Advanced Example

Decide whether system mode should track live platform brightness changes, whether theme changes animate, and how custom colors adapt. Persist the user's policy (`system/light/dark`), not only the current resolved brightness, so system mode can continue responding later.

# Under the Hood

`MaterialApp.themeMode` chooses between configured light/dark themes and system platform brightness. Changing it updates inherited theme data for descendants. Preference loading happens asynchronously, so apps may gate first display or accept a brief default transition.

# Performance Notes

- Load theme preference once at app initialization.
- Avoid rebuilding unrelated application services.
- Do not animate every color independently when theme data can supply it.
- Test both themes for contrast and assets.

# Best Practices

- Store named values, not enum indexes.
- Validate unknown stored values.
- Preserve system-following behavior.
- Centralize global theme state.
- Test startup restoration and dynamic platform changes.

# Common Mistakes

- Persisting only current brightness rather than user policy.
- Defaulting to a forced light theme without product reason.
- Using hard-coded colors that fail in dark mode.
- Flashing a default theme before loading.
- Duplicating theme state across screens.

# Debugging

- Inspect stored theme string.
- Verify `themeMode`, `theme`, and `darkTheme`.
- Toggle device brightness while system mode is selected.
- Check images, icons, dialogs, and system UI overlays in both modes.

# Interview Questions

1. What are the three `ThemeMode` values? System, light, and dark.
2. Why store a string rather than enum index? It is stable across enum ordering changes.
3. What does system mode do? Follows platform brightness.
4. Why configure both `theme` and `darkTheme`? The app needs complete style definitions for each mode.

# Try It Yourself

1. Add a preview sample showing cards and text in both modes.
2. Handle an unknown stored theme value.
3. Make the app default to system without a stored key.
4. Audit hard-coded colors.

# Challenge Project

Build a theme settings page with system/light/dark options, persistent restoration, and accessible contrast. Test it while changing the device brightness.

# Quick Revision Sheet

- Persist policy: `system`, `light`, or `dark`.
- Map stored strings safely; avoid enum indexes.
- Configure `theme`, `darkTheme`, and `themeMode`.
- Load once and expose global observable state.
- Audit contrast in both themes.

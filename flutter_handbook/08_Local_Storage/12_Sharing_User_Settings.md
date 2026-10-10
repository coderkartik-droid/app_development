# Introduction

User settings are small choices that influence how an app behaves or appears: theme, language, text size, and notification preferences. Sharing settings means making one authoritative settings state available to the widgets that need it.

# Why this concept exists

If each screen independently reads and writes a preference, state can drift. A settings repository plus app-level state makes changes visible immediately and persists them consistently.

# Real-world analogy

A household has one thermostat. Every room reads the same selected temperature rather than maintaining a conflicting private setting.

# Internal Working

A settings service reads persisted values during startup and exposes a typed settings model to the app. User edits update that model and persist through the service. Widgets rebuild from the shared state owner.

Keep preference persistence separate from the mechanism that broadcasts updates. A preferences plugin stores data; a state-management layer makes the current value observable. For account-level settings shared across devices, synchronize with a backend.

# Syntax

```dart
class UserSettings {
  const UserSettings({required this.darkMode, required this.localeCode});
  final bool darkMode;
  final String localeCode;
}
```

# Complete Example 1

This runnable example shows one settings page changing an app theme and saving the choice. Add `shared_preferences` to dependencies.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const SettingsApp());

class SettingsApp extends StatefulWidget {
  const SettingsApp({super.key});

  @override
  State<SettingsApp> createState() => _SettingsAppState();
}

class _SettingsAppState extends State<SettingsApp> {
  bool _dark = false;
  bool _loaded = false;

  @override
  void initState() {
    super.initState();
    _load();
  }

  Future<void> _load() async {
    final prefs = await SharedPreferences.getInstance();
    final value = prefs.getBool('settings.darkMode') ?? false;
    if (!mounted) return;
    setState(() {
      _dark = value;
      _loaded = true;
    });
  }

  Future<void> _changeTheme(bool value) async {
    final prefs = await SharedPreferences.getInstance();
    final ok = await prefs.setBool('settings.darkMode', value);
    if (!mounted) return;
    if (ok) setState(() => _dark = value);
  }

  @override
  Widget build(BuildContext context) => MaterialApp(
        theme: ThemeData.light(),
        darkTheme: ThemeData.dark(),
        themeMode: _dark ? ThemeMode.dark : ThemeMode.light,
        home: Scaffold(
          appBar: AppBar(title: const Text('User settings')),
          body: !_loaded
              ? const Center(child: CircularProgressIndicator())
              : SwitchListTile(
                  title: const Text('Dark theme'),
                  value: _dark,
                  onChanged: _changeTheme,
                ),
        ),
      );
}
```

### Line-by-line explanation

- The app-level `State` owns the shared theme value, so the root `MaterialApp` can consume it.
- `_load` restores the persisted setting and sets `_loaded` only after reading.
- `_changeTheme` writes the new value and updates app state after success.
- `MaterialApp.themeMode` selects the active theme.
- `SwitchListTile` is a labeled accessible control.

### Widget tree

```text
SettingsApp (shared state)
└── MaterialApp(themeMode from state)
    └── Scaffold
        └── SwitchListTile
```

### Execution flow

```text
Root initializes → load settings → MaterialApp reflects values
Toggle → persist setting → root state updates → all themed widgets rebuild
```

### ASCII phone preview

```text
+---------------------------+
|       User settings       |
+---------------------------+
| Dark theme           [on] |
+---------------------------+
```

### Example explanation

The setting is owned above the UI it affects. This means a theme change affects the whole app rather than only the settings screen. In a larger application, move loading and persistence into a repository/notifier instead of keeping plugin calls in the root widget.

If only the switch widget stores its own value, other screens may not update. If the preference write is omitted, the choice is lost after a restart.

**Common mistakes:** Avoid multiple widgets writing the same key independently. Define a single settings owner and typed model.

**Professional use case:** App-wide theme and display preferences.

# Complete Example 2

Share one setting object:

```dart
final settings = UserSettings(darkMode: true, localeCode: 'en');
```

Expose it through a state notifier/provider and persist changes through a repository. The object alone does not notify widgets or write to disk.

# Complete Example 3

Use a feature-scoped preference key:

```dart
await prefs.setBool('reader.showPageNumbers', false);
```

Namespacing distinguishes reader settings from unrelated settings and makes ownership clearer.

# Advanced Example

For settings synchronized to a user account, define precedence among defaults, device-specific choices, and server values. Handle offline edits and conflicts. Some settings, such as text size, may be device-specific; others, such as notification subscriptions, may be account-level.

# Under the Hood

The storage layer persists values; an app state holder notifies dependent widgets. `MaterialApp` responds to updated theme/locale configuration by rebuilding relevant inherited configuration and descendants.

# Performance Notes

- Load a settings model once rather than per widget.
- Avoid rebuilding unrelated app areas for a minor setting.
- Persist only changed values.
- Keep settings models immutable for predictable updates.

# Best Practices

- Centralize keys, defaults, validation, and migrations.
- Distinguish account settings from device settings.
- Expose typed state to the UI.
- Persist before reporting success or define optimistic rollback.
- Test startup hydration and live updates.

# Common Mistakes

- Duplicating preference reads across screens.
- Having several owners for one key.
- Updating local switch state but not app-wide state.
- Syncing device-only settings unintentionally.
- Ignoring failure during persistence.

# Debugging

- Verify the active state owner and preference key.
- Check when root theme/locale is rebuilt.
- Test both initial hydration and an in-session change.
- Inspect key ownership when duplicate settings appear.

# Interview Questions

1. Why combine repository and state management? One handles persistence, the other exposes observable current state.
2. Where should global theme state live? At or above the app configuration that consumes it.
3. Should every setting sync to an account? No; classify device-specific versus account-level.
4. How do you avoid conflicting writes? Centralize ownership in a settings repository.

# Try It Yourself

1. Add a locale choice alongside dark mode.
2. Make the settings model immutable.
3. Add validation for supported locale codes.
4. Test a failed write and define the UI response.

# Challenge Project

Build a settings repository and observable settings controller for theme, locale, and reader text scale. Include defaults, validation, live app updates, and tests for cold-start hydration.

# Quick Revision Sheet

- Settings need persistence plus observable app state.
- Centralize key ownership and defaults.
- Keep device and account settings distinct.
- Load once, expose typed state, and persist updates.
- Test hydration, live changes, and failures.

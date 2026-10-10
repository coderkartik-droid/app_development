# Introduction

“Sharing data between sessions” can mean two different things in Flutter:

1. Passing data between screens or routes during one running app session.
2. Keeping data across app launches, users, or devices.

These are not interchangeable. Route arguments and app state share in-memory data while the process is running. Local storage survives process restarts on that device. Cloud synchronization is needed to share data across devices or account sessions.

# Why this concept exists

Developers often reach for local storage to solve a screen-to-screen state problem. That can create unnecessary disk reads and stale duplicate state. Conversely, relying on a singleton or provider for information that must survive a cold restart loses data.

Choosing the right lifetime keeps data consistent and avoids accidental privacy leaks between accounts.

# Real-world analogy

Passing a value to another screen is like handing a note to a colleague in the same office. Saving a value locally is like filing the note in a cabinet for tomorrow. Sharing it with another office requires sending it through a trusted network.

# Internal Working

Route arguments are passed through navigation and exist in memory. State management tools such as `Provider`, `Riverpod`, or `Bloc` can expose shared in-memory state to multiple widgets. Their state is normally lost when the process ends unless explicitly hydrated from storage.

Persistent local data is read from preferences, a database, or another device store. Account-scoped data must be associated with an account ID and cleared or isolated on sign-out. If multiple accounts can be used, one shared unscoped key can show the wrong user's data.

Cross-device sharing needs a backend or platform sync mechanism. A local write alone cannot notify another device.

# Syntax

Pass a small route value directly:

```dart
Navigator.of(context).push(
  MaterialPageRoute<void>(
    builder: (_) => DetailsPage(itemId: item.id),
  ),
);
```

Use a stable identifier and let the destination load the current record from the repository when appropriate. This avoids passing a large stale object graph through navigation.

# Complete Example 1

This runnable app passes a selected counter value from one route to another. It demonstrates session-level sharing without disk persistence.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RouteDataApp());

class RouteDataApp extends StatelessWidget {
  const RouteDataApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(home: CounterPage());
  }
}

class CounterPage extends StatefulWidget {
  const CounterPage({super.key});

  @override
  State<CounterPage> createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Route data')),
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text('Current count: $_count'),
            const SizedBox(height: 16),
            FilledButton(
              onPressed: () => setState(() => _count++),
              child: const Text('Add one'),
            ),
            FilledButton(
              onPressed: () {
                Navigator.of(context).push(
                  MaterialPageRoute<void>(
                    builder: (_) => CountDetailsPage(count: _count),
                  ),
                );
              },
              child: const Text('Open details'),
            ),
          ],
        ),
      ),
    );
  }
}

class CountDetailsPage extends StatelessWidget {
  const CountDetailsPage({required this.count, super.key});

  final int count;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Details')),
      body: Center(child: Text('Received count: $count')),
    );
  }
}
```

### Line-by-line explanation

- `RouteDataApp` starts a Material app whose first route is `CounterPage`.
- `_count` is in-memory state owned by the counter page.
- The `Text` shows the current value.
- The increment button updates `_count` with `setState`.
- The details button pushes a new `MaterialPageRoute`.
- `CountDetailsPage` receives `count` through its constructor and displays it.
- Returning from details leaves the original route state alive in the navigation stack.

### Widget tree

```text
MaterialApp
└── CounterPage
    └── Scaffold
        └── Column
            ├── Text
            ├── FilledButton (increment)
            └── FilledButton (navigate)

Pushed route:
└── CountDetailsPage(count)
    └── Scaffold → Text
```

### Execution flow

```text
Counter state changes in memory
          |
          v
Push route with current count
          |
          v
Destination displays constructor argument
          |
          v
Pop returns to retained counter route
```

### ASCII phone preview

```text
+---------------------------+
|       Route data          |
+---------------------------+
| Current count: 3          |
| [       Add one       ]   |
| [    Open details     ]   |
+---------------------------+
```

### Example explanation

The route receives a snapshot of `_count` at navigation time. It is ideal for a small value that the destination only needs to display. No persistence package or disk I/O is necessary.

If the process is killed, the counter state is lost. If the destination needs live updates, pass a repository/state object or observe shared state rather than a snapshot. If the details screen should survive a restart, persist the underlying record and pass its ID.

**Common mistakes:** Passing a mutable model and assuming changes are automatically synchronized is fragile. Prefer immutable data or stable IDs and a clear state owner.

**Professional use case:** Pass record IDs or lightweight arguments between routes, while a repository manages the authoritative in-memory/persistent data.

# Complete Example 2

An in-memory shared service can expose current session data to several widgets. With a state management package, subscribe to changes rather than copying values into multiple unrelated states.

```dart
class SessionUser {
  const SessionUser({required this.id, required this.displayName});

  final String id;
  final String displayName;
}
```

This type alone does not share state; a provider/notifier/repository must own and expose the current instance. Persisting or refreshing the session is a separate responsibility.

**Flow:** authentication result updates session owner → subscribed widgets rebuild → sign-out clears state and account-scoped data.

# Complete Example 3

Persist a preference for the next launch:

```dart
final prefs = await SharedPreferences.getInstance();
await prefs.setString('preferredLanguage', 'fr');
```

Later, read it during initialization and choose a supported locale. Preferences survive ordinary process restarts on the same installation, but do not automatically follow the user to another device.

**Flow:** user selects locale → save locally → app restarts → read saved locale → configure localization.

If the app does not await the write, it may close before the operation completes. Do not store an entire account object as a casual preference if sensitive or frequently changing data is involved.

# Advanced Example

For account-scoped local data, include the authenticated account identifier in repository queries or storage keys, and clear sensitive account data on sign-out according to the product's retention rules. Ensure data for one account cannot appear momentarily while another account's data is loading.

For cross-device session sharing, authenticate with a backend and define server-side source of truth, token refresh, conflict handling, and offline policy. Never treat a local boolean such as `isLoggedIn` as proof that the server still accepts a session.

# Under the Hood

Navigation constructs and retains route objects in a `Navigator` stack. State management listeners notify subscribed widgets when in-memory values change. Persistence plugins write data through platform APIs or local files. The network is a separate boundary for syncing.

Each layer has different lifecycle and failure behavior: route state can disappear with navigation, in-memory providers disappear with process death, local files disappear with app-data removal, and remote records depend on account/service availability.

# Performance Notes

- Pass stable IDs rather than large object graphs.
- Avoid reading the same preference repeatedly in every widget build.
- Load records asynchronously through repositories.
- Keep account switches from rendering stale previous-account content.
- Use pagination for large shared datasets.

# Best Practices

- Identify whether the required lifetime is widget, route, app process, installation, account, or cross-device.
- Keep one authoritative owner for mutable state.
- Use route arguments for lightweight navigation context.
- Use repositories for persistent data access.
- Scope account data and clear or isolate it on sign-out.
- Treat server authentication as authoritative.

# Common Mistakes

- Using disk storage for state that only needs to cross one route.
- Assuming provider state persists after process death.
- Assuming preferences sync across devices.
- Passing stale mutable objects between routes.
- Sharing unscoped account data across logins.
- Treating a local login flag as a valid credential.

# Debugging

- Check which layer owns the value and how long that owner lives.
- Reproduce a cold process restart to test persistence.
- Switch accounts and verify no previous account data appears.
- Check whether a route argument is a snapshot or live observable state.
- Confirm cloud data is fetched or synchronized rather than expected from local preferences.

# Interview Questions

1. **How do you share data between two screens in one navigation flow?** Pass constructor/route arguments or use a shared state owner, depending on whether the value is a snapshot or live state.
2. **Does Provider persist data?** Not by itself; it manages in-memory state unless combined with persistence.
3. **Do local preferences sync across devices?** No, not automatically.
4. **What should a details route receive?** Often a stable record ID, especially when it should load current data.
5. **Why scope data to an account?** To prevent data leakage or stale display between user accounts.
6. **Is `isLoggedIn = true` sufficient authentication?** No; session validity must be established with trusted credentials and the backend.

# Try It Yourself

1. Pass an item ID to a details page and load its title from an in-memory repository.
2. Restart the app and compare route state with persisted state.
3. Add a simulated second account and test data isolation.
4. Decide which onboarding state belongs in preferences and which belongs on a server.

# Challenge Project

Create a profile flow where a list screen opens a detail editor. Use route arguments for the profile ID, a shared repository for current in-memory updates, local storage for an offline draft, and a clearly described backend sync boundary.

# Quick Revision Sheet

- Route arguments: lightweight data within navigation.
- State management: shared in-memory state.
- Local storage: survives process restart on a device installation.
- Secure store: small secrets.
- Backend: account/device synchronization and authoritative auth.
- Data lifetime determines the sharing mechanism.
- Scope account data and clear/isolate it on sign-out.

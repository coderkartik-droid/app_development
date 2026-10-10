# Introduction

Local storage means keeping data on the device so an app can use it again later, including when the app is closed or the device is temporarily offline. Examples include a theme preference, a draft note, a cached response, or a database of tasks.

“Local storage” is not one technology. Flutter apps commonly use:

- Preferences for a small set of simple settings.
- Key-value databases such as Hive for structured objects.
- SQLite, often through Drift, for relational data and queries.
- Secure storage backed by platform facilities for small secrets such as tokens.
- Files for documents, images, exports, and other large byte data.

Choosing the right store depends on the data shape, query needs, sensitivity, and recovery requirements.

# Why this concept exists

Without persistence, ordinary in-memory variables disappear when an app process ends. Users would lose preferences, drafts, and offline work every time the operating system closed the app.

Local storage makes apps resilient and responsive. It can provide immediate reads, offline access, and a durable place to stage edits before syncing. It also adds responsibilities: schema changes, corruption handling, privacy, backup behavior, and consistency with remote data.

# Real-world analogy

Think of an app as a workshop. In-memory state is a whiteboard: quick to use, but erased when the room closes. Local storage is a set of labeled drawers. A settings drawer suits small items, a filing cabinet suits searchable records, and a locked safe suits sensitive keys.

# Internal Working

Flutter widgets and Dart objects normally live in process memory. To persist data, the app serializes a value into a format the chosen platform store understands, then writes it through a plugin or platform API. Later, the app reads and decodes it back into Dart values.

Most storage calls are asynchronous because disk or platform access can take time. Await writes when a later operation depends on them, and show sensible loading or error states. Do not block the UI isolate with expensive parsing or large synchronous work.

Persistence is not the same as backup or synchronization. A local database may be removed when the app is uninstalled, excluded from device backups, or overwritten by another device's newer state. Decide whether data is private, backed up, synced, or disposable.

# Syntax

A simple key-value read/write flow, shown with the `shared_preferences` package:

```dart
final preferences = await SharedPreferences.getInstance();
await preferences.setBool('darkMode', true);
final darkMode = preferences.getBool('darkMode') ?? false;
```

Add the package to the Flutter app's dependency manifest before importing it. Preferences support simple values; they are not a relational database and should not be used to store passwords or large documents.

# Complete Example 1

This runnable Flutter app remembers whether the user selected dark mode. It uses `shared_preferences`; add the package with your package manager before running.

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

void main() => runApp(const StorageIntroApp());

class StorageIntroApp extends StatefulWidget {
  const StorageIntroApp({super.key});

  @override
  State<StorageIntroApp> createState() => _StorageIntroAppState();
}

class _StorageIntroAppState extends State<StorageIntroApp> {
  bool _darkMode = false;
  bool _loading = true;

  @override
  void initState() {
    super.initState();
    _loadPreference();
  }

  Future<void> _loadPreference() async {
    final preferences = await SharedPreferences.getInstance();
    final value = preferences.getBool('darkMode') ?? false;
    if (!mounted) return;
    setState(() {
      _darkMode = value;
      _loading = false;
    });
  }

  Future<void> _setDarkMode(bool value) async {
    final preferences = await SharedPreferences.getInstance();
    await preferences.setBool('darkMode', value);
    if (!mounted) return;
    setState(() => _darkMode = value);
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData.light(),
      darkTheme: ThemeData.dark(),
      themeMode: _darkMode ? ThemeMode.dark : ThemeMode.light,
      home: Scaffold(
        appBar: AppBar(title: const Text('Local storage')),
        body: Center(
          child: _loading
              ? const CircularProgressIndicator()
              : SwitchListTile(
                  title: const Text('Use dark mode'),
                  value: _darkMode,
                  onChanged: _setDarkMode,
                ),
        ),
      ),
    );
  }
}
```

### Line-by-line explanation

- The Material import supplies Flutter UI; the preferences import supplies the key-value store API.
- `main` starts the app. The root is stateful because its `themeMode` changes.
- `_darkMode` stores the value displayed by the UI; `_loading` distinguishes “not loaded yet” from the default light setting.
- `initState` starts the asynchronous read once when the state is created.
- `_loadPreference` obtains the store, reads the boolean, and uses `false` if the key is absent.
- `mounted` prevents a late asynchronous result from updating a page that has already been removed.
- `_setDarkMode` awaits the write before updating the UI, so a failed write does not show a success-shaped state.
- `MaterialApp` selects light or dark theme using the loaded preference.
- The loading indicator avoids briefly presenting a possibly incorrect saved theme.
- `SwitchListTile` provides a labeled, accessible toggle.

### Widget tree

```text
StorageIntroApp
└── MaterialApp
    └── Scaffold
        ├── AppBar
        └── Center
            ├── CircularProgressIndicator (while loading)
            └── SwitchListTile (after loading)
```

### Execution flow

```text
App starts -> initState -> read local key
                               |
                        value or default
                               v
                     update theme and switch
                               |
User toggles -> await persistent write -> update visible state
```

### ASCII phone preview

```text
+---------------------------+
|       Local storage       |
+---------------------------+
|                           |
| Use dark mode        [on] |
|                           |
+---------------------------+
```

### Example explanation

The stored preference is small and independent, so a preference store is appropriate. The app loads it before showing the control, then persists each change. This example intentionally lets storage failures surface rather than silently pretending a write succeeded.

If the initial read is removed, the app always starts with the in-memory default. If `await` is removed from the write, the UI may update before persistence completes. If the `mounted` check is removed, a slow read could call `setState` after disposal.

**Common mistakes:** Do not store credentials in preferences. Do not use a preferences store for a large or query-heavy collection. Do not assume local data is synchronized across devices.

**Professional use case:** Theme, onboarding completion, display density, and other small user-selected options are suitable preference data.

# Complete Example 2

For a collection of notes, model each note with an ID and content, then use a database such as SQLite or an object store. A record can be represented in Dart like this:

```dart
class Note {
  const Note({required this.id, required this.title, required this.body});

  final int id;
  final String title;
  final String body;
}
```

The class describes an in-memory object, not persistence by itself. A database adapter or serializer must map each field to stored columns or encoded data.

**Widget tree:** a notes screen generally uses `FutureBuilder`/state management around a list, with each database record rendered as a tile.  
**Flow:** query store → decode records → render list → user edits → write change → refresh the observed data.

If IDs are omitted, updates and deletes become ambiguous. If serialization is omitted, the database cannot reconstruct the Dart object after restart.

# Complete Example 3

A sensitive token belongs in a platform-backed secure store rather than ordinary preferences:

```dart
await secureStorage.write(key: 'accessToken', value: token);
final token = await secureStorage.read(key: 'accessToken');
```

The `secureStorage` object comes from a secure-storage package and requires platform setup. Secure storage is for small secrets, not bulk application data. A token still needs expiration, revocation, and server-side authorization.

**Flow:** authentication service returns token → secure write completes → later startup reads token → app validates session with service.

If the token is stored as a plain preference, it may be exposed by backups or device inspection more readily than intended. Encryption does not replace secure authentication design.

# Advanced Example

For offline-first behavior, treat local storage as the source of immediate UI data and a synchronization queue as the record of pending remote changes. Store a stable record ID, local modification time, remote revision if available, and sync status. Define conflict behavior before implementation: last-write-wins is simple but can silently discard edits.

Use schema migrations when persisted structure changes. Keep migrations deterministic and tested against old database versions. Decide which records can be recreated from a server and which represent irreplaceable user work.

# Under the Hood

Plugins bridge Dart APIs to native platform storage, or use a Dart implementation that manages files and database pages. The returned Dart values are decoded copies; mutating an object in memory does not automatically persist it unless the store's API is called.

Writes may be buffered or scheduled by the underlying system. An `await` confirms completion according to the package API, not necessarily that data is indestructible against sudden hardware failure. Read package guarantees when durability is critical.

# Performance Notes

- Keep preference values small and simple.
- Use indexes and targeted queries for databases.
- Avoid loading huge files or datasets synchronously on the UI isolate.
- Cache only data with a clear invalidation policy.
- Batch related writes when supported and safe.
- Measure startup time and database query performance on real devices.

# Best Practices

- Select storage by data shape, sensitivity, volume, and query needs.
- Centralize storage access behind a repository or service.
- Version persisted schemas and test migrations.
- Handle missing, corrupt, and outdated data.
- Distinguish local persistence from cloud backup and sync.
- Do not store secrets in plain preferences.
- Make storage failures observable and recoverable.

# Common Mistakes

- Expecting ordinary Dart variables to survive process death.
- Treating local data as a backup or remote sync.
- Putting large lists or relational records into preferences.
- Storing passwords, private keys, or tokens without appropriate protection.
- Ignoring schema upgrades and corrupt data.
- Silently swallowing write failures.
- Blocking the UI isolate with expensive data processing.

# Debugging

- Verify the exact key, type, and store instance used by both read and write.
- Log operation errors without logging secrets or sensitive user content.
- Inspect app data using supported platform tools in development builds.
- Test a cold restart, not only a hot rebuild.
- Reproduce migration behavior from an older schema fixture.
- Check whether app reinstall, backup restore, or clear-data operations affect the store as expected.

# Interview Questions

1. **What is local persistence?** Storing app data on the device so it survives beyond the current process.
2. **How do preferences differ from SQLite?** Preferences suit small independent key-value settings; SQLite supports relational records and queries.
3. **Is local storage the same as backup?** No; backup and restore behavior depends on platform and app configuration.
4. **Where should an access token go?** A platform-backed secure store, with proper session and server validation design.
5. **Why are storage APIs often asynchronous?** Disk and platform operations can take time and should not block UI work.
6. **What is a migration?** A controlled transformation from one persisted schema version to another.
7. **What does offline-first mean?** The app can operate on local data and later reconcile or synchronize remote changes.

# Try It Yourself

1. Save a preferred text size and restore it after a cold restart.
2. Write down which store best fits preferences, notes, images, and authentication tokens.
3. Simulate a missing value and choose a safe default.
4. Add error reporting to a storage write without exposing user data.
5. Define how a note conflict should be resolved between two devices.

# Challenge Project

Design the storage layer for a personal journal: theme preferences, searchable entries, image attachments, secure account tokens, and offline synchronization. Choose a store for each type, describe migrations and backup assumptions, and implement a repository boundary between UI and persistence.

# Quick Revision Sheet

- Preferences: small primitive settings.
- Hive/object stores: local structured objects.
- SQLite/Drift: relational records and queries.
- Secure storage: small secrets, protected by platform facilities.
- Files: large documents and binary content.
- Persistence is not automatically backup, encryption, or sync.
- Await writes, handle errors, version schemas, and test cold restarts.

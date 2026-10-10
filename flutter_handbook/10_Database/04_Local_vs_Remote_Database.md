# Introduction

A local database runs on the device and stores data in that device's application storage. A remote database runs on another machine and is reached through a service boundary. This chapter compares database placement only; it does not teach HTTP or API design.

# Why this concept exists

Placement changes offline behavior, latency, backup, sharing, security, and conflict handling. A local database offers fast access for one device; a remote database can coordinate shared records across devices when accessed through a properly secured service.

# Real-world analogy

A local database is a notebook in your bag: immediately available but private to that copy. A remote database is a shared office ledger: several people can coordinate, but access depends on a connection and permissions.

# Internal Working

With local storage, Flutter's database package communicates with an embedded engine and local file. With remote storage, the app talks to a remote system through an application boundary; clients should not receive unrestricted direct database credentials.

```text
Local:  Flutter -> SQLite engine -> device file
Remote: Flutter -> trusted service boundary -> remote database
```

# Syntax

```dart
// Local SQLite operation
final rows = await localDb.query('drafts');
```

```text
Remote access should be mediated by an authenticated service
that validates requests and enforces authorization.
```

# Complete Example 1

This runnable Flutter example writes and reads a local record using `sqflite`. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openJournal() async {
  final file = p.join(await getDatabasesPath(), 'placement_demo.db');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE journal (id INTEGER PRIMARY KEY, message TEXT NOT NULL)',
    );
  });
}

void main() => runApp(const MaterialApp(home: JournalPage()));

class JournalPage extends StatefulWidget {
  const JournalPage({super.key});
  @override
  State<JournalPage> createState() => _JournalPageState();
}

class _JournalPageState extends State<JournalPage> {
  late final Future<Database> _db = openJournal();
  String _message = 'Loading local record...';

  Future<void> _saveAndRead() async {
    final db = await _db;
    await db.insert('journal', {'id': 1, 'message': 'Stored on this device'},
        conflictAlgorithm: ConflictAlgorithm.replace);
    final rows = await db.query('journal', where: 'id = ?', whereArgs: [1]);
    if (mounted) {
      setState(() => _message = rows.single['message']! as String);
    }
  }

  @override
  void initState() {
    super.initState();
    _saveAndRead();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Local database')),
        body: Center(child: Text(_message)),
      );
}
```

**Line-by-line guide:** Imports set up Flutter, path handling, and SQLite. `openJournal` chooses the app-private database directory and creates a table once. The integer primary key identifies one journal row; its message cannot be null. The page state retains a single database future. `_saveAndRead` inserts or replaces ID 1, queries exactly that ID with a bound argument, and displays the value if the page still exists. `initState` starts this operation once. The widget tree is `MaterialApp -> JournalPage -> Scaffold -> Center -> Text`.

**Database flow:** `Dart insert -> local SQLite file -> parameterized query -> row map -> state -> Text`.

**Execution:** The sample writes immediately and reads the persisted record. Removing the conflict policy makes repeated runs attempt a duplicate primary key; removing the query means the UI has no database result to display.

```text
+----------------------+
| Local database       |
|                      |
| Stored on this device|
+----------------------+
```

**Common mistakes:** Assuming local records automatically synchronize, putting secrets in plaintext fields, or placing a remote database credential in the client.

**Professional use case:** Drafts and offline-first working data are local; shared authoritative data is owned by a remote service.

# Complete Example 2

A local-first read path checks the on-device database before showing content. If a refresh is later available, update the local copy and notify state. The database remains the UI's read source, keeping rendering independent of availability. This pattern requires a separate, well-defined synchronization design; local storage alone does not sync.

# Complete Example 3

For data shared between devices, the app should submit an operation to a trusted service that validates the caller and writes to the remote database. The service, not the mobile app, holds database credentials and enforces authorization. Do not connect a production app directly to a database port with administrator credentials.

# Advanced Example

Offline synchronization needs stable record IDs, version or conflict metadata, retryable pending operations, deletion tombstones, and an explicit conflict policy (server wins, last-write-wins, or merge). Transactions can keep local queue updates atomic; synchronization must be idempotent to tolerate retries.

# Under the Hood

Local database latency is generally bounded by device storage and scheduling. Remote access adds network and service availability. A remote database is not equivalent to a remote database API; a secure service boundary controls validation, authorization, and schema access.

# Performance Notes

Use local persistence to avoid repeated slow reads and support offline workflows. Set limits on local cache growth. Remote systems need pagination and careful request patterns, but do not store all remote data indefinitely without a retention plan.

# Best Practices

- Classify each datum as device-only, shared, or synchronized.
- Keep credentials and authorization enforcement server-side.
- Document offline and conflict behavior.
- Encrypt or exclude sensitive local data according to threat model.
- Handle local migrations and remote schema evolution separately.

# Common Mistakes

- Calling local data “synced” without a sync process.
- Treating a database file as an access-control boundary.
- Exposing remote credentials in a client bundle.
- Failing to define which copy is authoritative.

# Debugging

For local issues inspect file path, schema version, and actual rows. For remote issues trace service-side authorization and database logs without logging private payloads. Test airplane mode and reconnection behavior.

# Interview Questions

1. **Why use a local database?** Fast, durable device storage and offline access.
2. **Why use a remote database?** Shared or centrally governed state across clients.
3. **Does local storage synchronize automatically?** No.
4. **Why not embed remote DB credentials in Flutter?** Client binaries can be inspected; credentials would be exposed.

# Try It Yourself

For a notes app, classify title, draft body, account ID, and sync timestamp as local-only, shared, or synchronized. Explain the offline behavior for each.

# Challenge Project

Design a local-first inventory view with a local database and a synchronization boundary. Document conflict behavior and what the user sees offline.

# Quick Revision Sheet

- Local = device file and offline availability.
- Remote = shared system reached through a trusted service.
- Offline cache is not synchronization.
- Define data authority, conflicts, and retention.
- Never ship unrestricted database credentials in a mobile client.

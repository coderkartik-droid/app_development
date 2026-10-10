# Introduction

A one-to-one relationship means each record in one table is associated with at most one record in another. For example, a user profile extension can have one unique row for each user.

# Why this concept exists

Separating optional or sensitive attributes from a core record can keep the main table focused. A unique foreign key enforces that no two profile rows belong to the same user.

# Real-world analogy

Each employee has one assigned locker record, and each locker record belongs to exactly one employee.

# Internal Working

The child table stores a foreign key referencing the parent primary key. Adding a `UNIQUE` constraint to that foreign key enforces one-to-one cardinality; a foreign key enforces that the referenced parent exists.

```text
users (id PK) 1 -------- 0..1 profiles (user_id FK UNIQUE)
```

# Syntax

```sql
CREATE TABLE profiles (
  user_id INTEGER PRIMARY KEY REFERENCES users(id),
  display_name TEXT NOT NULL
);
```

# Complete Example 1

This runnable app inserts a user and exactly one profile, then joins them for display. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openProfileDb() async {
  final file = p.join(await getDatabasesPath(), 'one_to_one_demo.db');
  return openDatabase(file, version: 1, onConfigure: (db) async {
    await db.execute('PRAGMA foreign_keys = ON');
  }, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE users (id INTEGER PRIMARY KEY AUTOINCREMENT, email TEXT NOT NULL UNIQUE)',
    );
    await db.execute('''
      CREATE TABLE profiles (
        user_id INTEGER PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
        display_name TEXT NOT NULL
      )
    ''');
  });
}

void main() => runApp(const MaterialApp(home: ProfilePage()));

class ProfilePage extends StatefulWidget {
  const ProfilePage({super.key});
  @override
  State<ProfilePage> createState() => _ProfilePageState();
}

class _ProfilePageState extends State<ProfilePage> {
  late final Future<Database> _db = openProfileDb();
  String _text = 'Loading profile...';

  Future<void> _load() async {
    final db = await _db;
    await db.transaction((txn) async {
      final userId = await txn.insert('users', {'email': 'reader@example.test'});
      await txn.insert('profiles', {
        'user_id': userId,
        'display_name': 'Reader',
      });
    });
    final rows = await db.rawQuery('''
      SELECT users.email, profiles.display_name
      FROM users JOIN profiles ON profiles.user_id = users.id
    ''');
    if (mounted && rows.isNotEmpty) {
      setState(() => _text = '${rows.first['display_name']} - ${rows.first['email']}');
    }
  }

  @override
  void initState() {
    super.initState();
    _load();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('One-to-one profile')),
        body: Center(child: Text(_text)),
      );
}
```

**Line-by-line guide:** Imports set up UI, paths, and SQLite. `openProfileDb` enables foreign keys on the connection and creates a user table with unique email plus a profile table whose primary key is also its foreign key. That shared key enforces at most one profile per user. `ON DELETE CASCADE` removes the profile with its user. `_load` transactionally inserts the parent then child using the generated ID. A join returns both values. The mounted check protects UI state. `initState` starts the flow; `build` displays the joined result. Widget tree: `MaterialApp -> ProfilePage -> Scaffold -> AppBar + Center/Text`.

**Database flow:** `INSERT user -> get user ID -> INSERT profile with same ID -> JOIN -> UI`.

**Execution:** The transaction ensures both rows are written together. Removing `PRIMARY KEY`/`UNIQUE` from `profiles.user_id` permits multiple profiles; removing foreign-key configuration may disable enforcement.

```text
+----------------------+
| One-to-one profile   |
|                      |
| Reader - reader@...  |
+----------------------+
```

**Common mistakes:** Assuming a foreign key alone enforces one-to-one, forgetting SQLite foreign-key enforcement, and cascading deletes without considering product requirements.

**Professional use case:** Optional profile details kept separate from a core record.

# Complete Example 2

Find users that do not yet have a profile:

```sql
SELECT users.id, users.email
FROM users
LEFT JOIN profiles ON profiles.user_id = users.id
WHERE profiles.user_id IS NULL;
```

The left join preserves every user; null on the child side indicates no profile. An inner join would hide users without profiles.

# Complete Example 3

Enforce the relation with a unique foreign key:

```sql
CREATE TABLE profiles (
  id INTEGER PRIMARY KEY,
  user_id INTEGER NOT NULL UNIQUE REFERENCES users(id)
);
```

The separate profile ID may be useful when other tables refer to the profile. The `UNIQUE` constraint remains essential for one-to-one cardinality.

# Advanced Example

Choose cascade, restrict, or set-null delete behavior deliberately. If the child is optional and must survive parent removal, `ON DELETE SET NULL` requires a nullable foreign key. Test every deletion policy in a transaction.

# Under the Hood

A foreign key checks referential validity; uniqueness is a separate index-backed invariant. One-to-one is therefore modeled by a foreign key plus uniqueness, or a shared primary/foreign key.

# Performance Notes

Primary and unique keys are indexed. Index the child foreign key when it is not already a primary/unique key and joins/deletes need it.

# Best Practices

- Use the foreign key as a primary key for a strict shared-identity extension.
- Add `UNIQUE` when using a separate child key.
- Enable and verify foreign-key enforcement.
- Make optionality and delete behavior explicit.

# Common Mistakes

- Calling a foreign-key-only relationship one-to-one.
- Forgetting `NOT NULL` when the child must always have a parent.
- Joining without matching keys.
- Assuming cascades work when foreign keys are off.

# Debugging

Run `PRAGMA foreign_keys`, inspect constraints and indexes, and test duplicate child keys. Use `PRAGMA foreign_key_check` to locate violations.

# Interview Questions

1. **How enforce one-to-one?** Foreign key plus unique constraint, often a shared primary key.
2. **What is a foreign key?** A reference to a key in another table.
3. **Why enable `PRAGMA foreign_keys`?** SQLite enforcement may need explicit per-connection enablement.
4. **What does cascade do?** Applies a configured child action when a parent is deleted/updated.

# Try It Yourself

Add an optional address table and decide whether users may have zero or exactly one address.

# Challenge Project

Model a device and its unique settings record; enforce one-to-one cardinality and test parent deletion.

# Quick Revision Sheet

- One-to-one = foreign key + uniqueness.
- Shared PK/FK is a compact strict pattern.
- Enable foreign keys and choose delete behavior.
- Left join reveals missing optional children.

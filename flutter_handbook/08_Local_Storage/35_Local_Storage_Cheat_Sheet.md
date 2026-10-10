# Introduction

This cheat sheet summarizes the storage choices, APIs, and design checks covered in the Local Storage handbook. Use it to orient yourself, then consult the relevant full chapter for package-specific details.

# Why this concept exists

Storage choices are easier when the data requirement is visible at a glance. A concise reference helps select the right tool without confusing persistence, security, caching, and synchronization.

# Real-world analogy

This is the label on a toolbox drawer: it helps find the right tool quickly, but the repair still requires understanding how to use it safely.

# Internal Working

Every persisted value has a lifetime, owner, format, and failure mode. Decide who writes it, who reads it, how it evolves, who can access it, and what happens when it is missing or corrupt.

# Syntax

## Storage selection

| Data | Good starting point |
|---|---|
| Theme, small flags, short settings | `shared_preferences` |
| Object/key-value records | Hive or comparable object store |
| Relational/query-heavy records | SQLite or Drift |
| Small credentials | Secure storage |
| Images/documents/large bytes | App files; store metadata in DB |
| Remote cache | Local store plus freshness/eviction policy |

## Key calls

```dart
await prefs.setBool('settings.darkMode', true);
final value = prefs.getBool('settings.darkMode') ?? false;
await prefs.remove('settings.darkMode');
```

# Complete Example 1

A tiny preference read/write:

```dart
Future<bool> saveThemePreference(bool dark) async {
  final prefs = await SharedPreferences.getInstance();
  return prefs.setBool('settings.darkMode', dark);
}
```

### Line-by-line explanation

- The function accepts the desired boolean state.
- It asynchronously obtains the preferences object.
- It returns the setter's result so callers can decide whether to show success.
- The key is namespaced by feature.

### Widget tree

No widget is required; the function belongs in a settings repository called by app state.

### Execution flow

```text
UI choice → repository → preferences setter → success/failure result
```

### ASCII phone preview

```text
+---------------------------+
| Settings                  |
| Dark mode           [on]  |
+---------------------------+
```

### Example explanation

This is suitable for a non-sensitive primitive. It does not guarantee cloud sync, encryption, or database transaction semantics. If the return value is discarded, the caller cannot distinguish a reported unsuccessful write.

**Common mistakes:** Never use this for passwords, large records, or high-frequency transactional data.

**Professional use case:** Persist a user-selected appearance preference.

# Complete Example 2

A parameterized database query:

```sql
SELECT id, title FROM notes WHERE id = ?;
```

Bind values through the database API. Never concatenate untrusted values into SQL.

# Complete Example 3

A secure credential access:

```dart
await secureStorage.write(key: 'refreshToken', value: token);
```

Validate/refresh the token with the authentication service and delete it on sign-out. Secure storage does not itself authenticate the user.

# Advanced Example

## Production checklist

- Store choice matches shape, size, query, and sensitivity.
- Keys, IDs, and schema versions are stable.
- Migrations are tested against old persisted data.
- Reads distinguish loading, missing, invalid, and failed.
- Writes are awaited and errors are visible.
- Cache freshness and eviction are defined.
- Offline sync has retry, idempotency, and conflict policy.
- Account switching cannot expose prior-user data.
- Backups, deletion, and retention are documented.
- Secrets and personal data do not leak to logs.

# Under the Hood

Preferences encode primitives, object stores serialize records, SQLite stores relational pages, secure storage delegates to OS credential facilities, and files store bytes. Repositories provide app-level ownership across these storage mechanisms.

# Performance Notes

- Query only needed data; index measured access paths.
- Batch database operations.
- Keep media out of preference/object blobs.
- Avoid repeated storage initialization.
- Profile realistic cold starts and data sizes.

# Best Practices

- Keep persistence behind typed repositories.
- Use parameter binding.
- Version schemas and test migrations.
- Validate persisted values.
- Treat persistence, backup, encryption, and sync as distinct properties.
- Provide deletion and recovery strategies.

# Common Mistakes

- Preferences for everything.
- Plaintext tokens or passwords.
- No schema migration.
- Silent catch-and-default behavior.
- In-memory demo mistaken for persistence.
- No account isolation or offline conflict policy.

# Debugging

- Trace the exact key/table/box/path.
- Test cold process start.
- Verify schema and adapter registration.
- Use sanitized logs.
- Test upgrades, offline mode, and failures.

# Interview Questions

1. Which store fits small settings? Preferences.
2. Which store fits relational queries? SQLite/Drift.
3. Where store secrets? Platform-backed secure storage.
4. Where store large images? Files, with metadata in a database.
5. What does offline-first require? Local durable data plus sync and conflict design.
6. Is local storage a backup? Not automatically.
7. How prevent SQL injection? Bound parameters.
8. Why use migrations? Persisted data must evolve with app schemas.

# Try It Yourself

1. Classify app data by store and sensitivity.
2. Create a cold-start persistence test.
3. Design a migration and rollback strategy.
4. Audit logs for secret exposure.

# Challenge Project

Design a local data layer for a small offline-capable app, selecting stores for preferences, records, media, and tokens. Implement repository interfaces, tests, migration notes, and a documented sync/deletion policy.

# Quick Revision Sheet

- Preferences: small values.
- Hive: object/key-value storage.
- SQLite/Drift: relational queries and transactions.
- Secure storage: small secrets.
- Files: large bytes.
- Cache: freshness + eviction.
- Offline-first: durable local changes + safe sync.
- Migrate, validate, test cold starts, and handle failures explicitly.

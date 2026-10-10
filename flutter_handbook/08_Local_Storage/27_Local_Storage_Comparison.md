# Introduction

Flutter apps can choose among preference stores, Hive, SQLite/Drift, files, and secure storage. Each solves a different data problem; no one choice is best for all values.

# Why this concept exists

Choosing storage by popularity rather than data requirements can lead to hard-to-query blobs, weak secret protection, or unnecessary database complexity.

# Real-world analogy

Use a pocket notebook for a few reminders, a filing system for searchable records, a locked key cabinet for credentials, and a shelf for large documents. The container should match the contents.

# Internal Working

Storage options differ in serialization, query capability, transaction guarantees, encryption, lifecycle, and platform dependencies. Compare the data shape and failure model first, then package maturity and team expertise.

# Syntax

| Need | Typical choice | Strength |
|---|---|---|
| Small primitive settings | `shared_preferences` | Simple key-value |
| Typed local objects, key lookup | Hive or similar object store | Object-oriented access |
| Relational data and queries | SQLite / Drift | SQL, indexes, transactions |
| Small credentials | Secure storage | Platform credential facilities |
| Large binary assets | App files/cache | Efficient byte storage |

# Complete Example 1

A small theme choice fits preferences:

```dart
await prefs.setString('themeMode', 'dark');
```

This is concise and suitable for a single non-sensitive setting. It is not appropriate for a large user profile database.

### Line-by-line explanation

- `prefs` is an initialized preferences store.
- `setString` writes one named preference.
- The key is namespaced, and the value is a stable named mode.

### Widget tree

No widget is required; app-level state reads this value and supplies `ThemeMode`.

### Execution flow

```text
Theme choice → preference repository → platform key-value store
```

### ASCII phone preview

```text
+---------------------------+
| Theme                     |
| ( ) System                |
| ( ) Light                 |
| (*) Dark                  |
+---------------------------+
```

### Example explanation

Preferences are a good match for one small setting. If the app needs relationships, filtering, or transaction guarantees, use a database. If this line is not awaited, the caller cannot know whether the operation completed before moving on.

**Common mistakes:** Do not store a real password in a preference string.

**Professional use case:** App display preferences.

# Complete Example 2

A relational note record belongs in SQLite/Drift:

```sql
SELECT id, title FROM notes WHERE updated_at > ? ORDER BY updated_at DESC;
```

SQL expresses filtering and ordering efficiently, especially with indexes. A preferences blob would require loading and scanning the whole collection in Dart.

# Complete Example 3

A refresh token belongs in platform-backed secure storage:

```dart
await secureStorage.write(key: 'refreshToken', value: token);
```

The token still needs validation and rotation; secure storage is not an authentication system.

# Advanced Example

Many apps use more than one store: SQLite for records, files for attachments, secure storage for credentials, and preferences for settings. Define repositories and consistency rules across them. A multi-store operation is rarely atomic, so make partial failure recoverable.

# Under the Hood

Preferences store small typed values; object stores serialize records; SQLite persists relational pages; secure storage delegates to OS facilities; files store bytes. Their APIs and durability guarantees differ.

# Performance Notes

- Benchmark representative data volumes.
- Avoid database overhead for one simple setting.
- Avoid preference scans for large collections.
- Store binary media as files.
- Avoid expensive serialization on the UI isolate.

# Best Practices

- Decide by data shape, query patterns, sensitivity, and offline requirements.
- Separate storage behind repositories.
- Review maintenance, platform support, and migration tools.
- Document backup and sync expectations.
- Test realistic scale and upgrade behavior.

# Common Mistakes

- Choosing one store for every kind of data.
- Using preferences as an object database.
- Using secure storage for large files.
- Assuming a database is encrypted automatically.
- Ignoring package version and migration support.

# Debugging

- Identify the exact store owning the value.
- Verify its encoding, type, and schema.
- Test cold restart and upgrade paths.
- Profile query/write behavior at realistic volume.
- Check platform-specific support and setup.

# Interview Questions

1. When choose preferences? Small primitive settings.
2. When choose SQLite? Relational data, complex queries, constraints, and transactions.
3. When choose secure storage? Small secrets requiring OS-backed protection.
4. Where store images? App files/cache, with metadata/path in a database.
5. Does Hive provide relational joins? No.

# Try It Yourself

1. Classify ten data items from an app by store.
2. Explain why a notes collection should not be a preference string list.
3. Identify which data requires encryption.
4. Compare migration tools for candidate libraries.

# Challenge Project

For a travel app, design storage for settings, itinerary entries, map tiles, offline photos, and authentication tokens. Choose a store for each and define retention, encryption, sync, and migration policies.

# Quick Revision Sheet

- Preferences: simple values.
- Hive: object/key-value access.
- SQLite/Drift: relational queries and transactions.
- Secure storage: small secrets.
- Files: large bytes.
- Apps commonly combine stores behind repositories.
- Evaluate migration, backup, sync, and threat model.

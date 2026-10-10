# Introduction

A transaction groups database operations into one logical unit. Either every operation succeeds and commits, or a failure rolls back the group so the database is not left half-updated.

# Why this concept exists

Business actions often touch several rows or tables. Without a transaction, a crash or error between writes can produce inconsistent state.

# Real-world analogy

A bank transfer is not “subtract money” followed later by “maybe add money.” Both sides must happen together or neither should happen.

# Internal Working

SQLite starts a transaction, applies statements, then commits on successful completion. If an exception escapes the transaction callback, the package rolls back. Reads inside the callback should use the transaction handle provided.

```text
BEGIN -> statement A -> statement B -> COMMIT
                    \-> error -> ROLLBACK
```

# Syntax

```dart
await db.transaction((txn) async {
  await txn.insert('accounts', debitValues);
  await txn.insert('ledger', entryValues);
});
```

# Complete Example 1

This runnable app atomically inserts a purchase and its audit event. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openPurchaseDb() async {
  final file = p.join(await getDatabasesPath(), 'transactions_demo.db');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE purchases (id INTEGER PRIMARY KEY AUTOINCREMENT, item TEXT NOT NULL)',
    );
    await db.execute(
      'CREATE TABLE events (id INTEGER PRIMARY KEY AUTOINCREMENT, purchase_id INTEGER NOT NULL, action TEXT NOT NULL)',
    );
  });
}

void main() => runApp(const MaterialApp(home: TransactionPage()));

class TransactionPage extends StatefulWidget {
  const TransactionPage({super.key});
  @override
  State<TransactionPage> createState() => _TransactionPageState();
}

class _TransactionPageState extends State<TransactionPage> {
  late final Future<Database> _db = openPurchaseDb();
  String _status = 'No purchase saved';

  Future<void> _save() async {
    final db = await _db;
    await db.transaction((txn) async {
      final purchaseId = await txn.insert('purchases', {'item': 'Desk lamp'});
      await txn.insert('events', {
        'purchase_id': purchaseId,
        'action': 'created',
      });
    });
    final count = Sqflite.firstIntValue(
          await db.rawQuery('SELECT COUNT(*) FROM events'),
        ) ??
        0;
    if (mounted) setState(() => _status = 'Purchase saved; $count event(s)');
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Atomic write')),
        body: Center(child: Text(_status)),
        floatingActionButton: FloatingActionButton(
          onPressed: _save,
          child: const Icon(Icons.save),
        ),
      );
}
```

**Line-by-line guide:** Imports provide the UI and database APIs. `openPurchaseDb` creates two tables: a purchase and its event. The page stores the database future and visible status. `_save` opens a transaction, inserts the purchase, uses its generated ID for the event, then exits the callback so the transaction can commit. It queries the event count only after commit. `mounted` protects state. The button invokes the operation. Widget tree: `MaterialApp -> TransactionPage -> Scaffold -> AppBar + Center/Text + FAB`.

**Database flow:** `BEGIN -> insert purchase -> insert event with generated ID -> COMMIT -> count -> UI`.

**Execution:** A successful tap produces both rows. If the event insert fails, the purchase insertion is rolled back. If transaction wrapping is removed, the purchase can remain without its event.

```text
+----------------------+
| Atomic write         |
|                      |
| Purchase saved; 1    |
|                 [S]  |
+----------------------+
```

**Common mistakes:** Using `db` instead of `txn` within the callback, catching an error and returning success, or putting slow non-database work inside a transaction.

**Professional use case:** Order creation plus line items, balance transfers, or inventory decrement plus audit record.

# Complete Example 2

A conditional update can check a balance and deduct within one transaction:

```dart
await db.transaction((txn) async {
  final rows = await txn.query(
    'accounts',
    columns: ['balance_cents'],
    where: 'id = ?',
    whereArgs: [accountId],
  );
  if (rows.isEmpty || (rows.single['balance_cents']! as int) < amountCents) {
    throw StateError('Insufficient balance');
  }
  await txn.update(
    'accounts',
    {'balance_cents': (rows.single['balance_cents']! as int) - amountCents},
    where: 'id = ?',
    whereArgs: [accountId],
  );
});
```

The read and write share the transaction boundary. The thrown error aborts the callback and rolls back. For financial invariants, prefer a database-enforced conditional update and rigorously test concurrency.

# Complete Example 3

Use a transaction for a batch of seed rows:

```dart
await db.transaction((txn) async {
  for (final name in ['Food', 'Travel', 'Home']) {
    await txn.insert('categories', {'name': name});
  }
});
```

This prevents a partially seeded category set. For very large imports, use batch APIs inside an appropriate transaction and report progress outside the critical transaction.

# Advanced Example

Transactions provide atomicity and isolation within SQLite's supported connection model. Keep transaction callbacks short, deterministic, and database-only. Never await a user dialog or remote operation while a transaction is open.

# Under the Hood

SQLite uses journaling or write-ahead logging to recover from interrupted writes. Commit marks changes as durable according to the configured durability settings; rollback discards uncommitted changes.

# Performance Notes

Grouping many writes can significantly reduce overhead. Long transactions hold locks longer and can delay other work, so bound their duration and size.

# Best Practices

- Include all invariant-related writes in one transaction.
- Use the transaction handle in its callback.
- Let failures escape when rollback is required.
- Keep network, UI, and expensive computation outside the transaction.

# Common Mistakes

- Assuming multiple awaited statements are automatically atomic.
- Swallowing errors inside the callback.
- Nested transactions without understanding package semantics.
- Returning success before commit completes.

# Debugging

Force a controlled failure between writes and assert no partial records remain. Log transaction operation names and verify counts after completion.

# Interview Questions

1. **What is atomicity?** All operations in a transaction occur together or none do.
2. **How does an error cause rollback?** It must propagate out of the transaction callback.
3. **Why use `txn` rather than `db` inside?** The transaction handle participates in the active transaction.
4. **Should remote work occur inside a transaction?** No; it prolongs locks and cannot share SQLite's atomic boundary.

# Try It Yourself

Insert a parent and two child rows; intentionally fail the second child insert and verify the parent was rolled back.

# Challenge Project

Implement an inventory purchase that inserts an order, order lines, and stock decrements atomically, with tests for rollback.

# Quick Revision Sheet

- Transactions provide atomic multi-statement changes.
- Success commits; propagated failure rolls back.
- Use `txn` within the callback.
- Keep the callback short and database-only.

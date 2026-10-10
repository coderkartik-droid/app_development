# Introduction

An inventory system tracks products and the quantity available. A production-quality design records stock movements so that current balance can be reconciled and audited.

# Why this concept exists

Overwriting a quantity loses the history of how it changed. Recording receipts and issues provides traceability and lets the app compute or verify stock.

# Real-world analogy

Inventory is a storeroom ledger: each delivery adds units and each issue removes units, leaving a balance that can be counted and checked.

# Internal Working

Products have stable IDs; stock movements reference a product and carry signed quantity and time. A transaction inserts the movement and updates any materialized balance together.

```text
product -> stock movement rows -> SUM(delta) -> current quantity
                         transaction -> consistent update
```

# Syntax

```sql
SELECT products.id, products.name, COALESCE(SUM(movements.delta), 0) AS stock
FROM products
LEFT JOIN movements ON movements.product_id = products.id
GROUP BY products.id;
```

# Complete Example 1

This runnable Flutter example records a stock receipt and computes the on-hand quantity. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openInventoryDb() async {
  final file = p.join(await getDatabasesPath(), 'inventory_demo.db');
  return openDatabase(file, version: 1, onConfigure: (db) async {
    await db.execute('PRAGMA foreign_keys = ON');
  }, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE products (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL UNIQUE)',
    );
    await db.execute('''
      CREATE TABLE movements (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        product_id INTEGER NOT NULL REFERENCES products(id),
        delta INTEGER NOT NULL CHECK (delta != 0),
        created_at_ms INTEGER NOT NULL
      )
    ''');
  });
}

void main() => runApp(const MaterialApp(home: InventoryPage()));

class InventoryPage extends StatefulWidget {
  const InventoryPage({super.key});
  @override
  State<InventoryPage> createState() => _InventoryPageState();
}

class _InventoryPageState extends State<InventoryPage> {
  late final Future<Database> _db = openInventoryDb();
  String _stock = 'Loading';

  Future<void> _receiveStock() async {
    final db = await _db;
    await db.transaction((txn) async {
      final productId = await txn.insert(
        'products',
        {'name': 'Notebook'},
        conflictAlgorithm: ConflictAlgorithm.ignore,
      );
      final rows = await txn.query(
        'products',
        columns: ['id'],
        where: 'name = ?',
        whereArgs: ['Notebook'],
      );
      final id = rows.single['id']! as int;
      await txn.insert('movements', {
        'product_id': id,
        'delta': 12,
        'created_at_ms': DateTime.now().millisecondsSinceEpoch,
      });
      if (productId == 0) {
        debugPrint('Existing product reused');
      }
    });
    final result = await db.rawQuery(
      'SELECT COALESCE(SUM(delta), 0) AS stock FROM movements',
    );
    if (mounted) setState(() => _stock = '${result.single['stock']} units');
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Inventory')),
        body: Center(child: Text('Notebook: $_stock')),
        floatingActionButton: FloatingActionButton(
          onPressed: _receiveStock,
          child: const Icon(Icons.add),
        ),
      );
}
```

**Line-by-line guide:** Imports provide Flutter, path, and SQLite. `openInventoryDb` enables foreign keys and creates unique products plus movement rows. `delta` cannot be zero; positive values represent receipts and negative values issues. `_receiveStock` uses a transaction to insert or reuse a product, look up its ID, and write a movement. It then aggregates the balance and updates state. The FAB repeats the receipt. Widget tree: `MaterialApp -> InventoryPage -> Scaffold -> AppBar + Center/Text + FAB`.

**Database flow:** `product lookup/create -> movement +12 in transaction -> SUM(delta) -> state`.

**Execution:** First press creates the product and adds 12 units; each press adds another receipt. The sample has no separate quantity cache. If movement insertion fails, transaction rollback prevents an inconsistent product operation. For a real unique-conflict flow, verify the package's exact insert return semantics and prefer a deliberate upsert/query helper.

```text
+----------------------+
| Inventory            |
|                      |
| Notebook: 12 units   |
|                 [+]  |
+----------------------+
```

**Common mistakes:** Allowing stock to go negative without a policy, treating movement history as editable snapshots, and updating balance separately from its movement.

**Professional use case:** Offline stock counts and auditable stock-in/stock-out history.

# Complete Example 2

Record an issue as a negative `delta`, but validate available quantity in the same transaction. A check performed only in UI can race with another operation. Define whether backorders/negative stock are allowed.

# Complete Example 3

Add a low-stock query that groups movement deltas per product and filters balances below a reorder threshold. Use a `LEFT JOIN` so products with no movements appear with zero stock.

# Advanced Example

For large movement history, maintain a materialized balance carefully in the same transaction as each movement; periodically reconcile it against `SUM(delta)`. Add indexes for product/time queries and test repair procedures.

# Under the Hood

Movement rows are an append-oriented ledger. Aggregation reconstructs balance. A transaction keeps ledger and any cached balance consistent, while constraints protect product references and delta shape.

# Performance Notes

Aggregating all movement history can become costly. Partition/filter by product or time and consider a transactionally maintained balance after measuring.

# Best Practices

- Use immutable movement records where possible.
- Record signed quantity, timestamp, and reason.
- Make movement and balance updates atomic.
- Reconcile cached quantity to movement history.

# Common Mistakes

- Overwriting stock without audit history.
- Updating quantity outside a transaction.
- Failing to validate issue quantities.
- Using a cascading delete that erases required audit history.

# Debugging

Compare current stock to `SUM(delta)` for a product. Inspect the transaction's last movement and check foreign-key violations.

# Interview Questions

1. **Why store movements instead of only current quantity?** Auditability and reconciliation.
2. **How prevent partial stock changes?** Transactionally write movement and balance.
3. **How calculate on-hand quantity?** Sum signed movement deltas.
4. **Why might a cached balance exist?** To improve reads when history aggregation becomes expensive.

# Try It Yourself

Add stock issue, low-stock threshold, and a transaction that rejects an issue larger than current stock.

# Challenge Project

Build an inventory app with product CRUD, receipts, issues, movement history, low-stock report, migrations, and reconciliation tests.

# Quick Revision Sheet

- Product records define items; movement rows explain quantity changes.
- Signed deltas can calculate on-hand stock.
- Enforce foreign keys and positive/negative policy.
- Keep movement and cached balance atomic.

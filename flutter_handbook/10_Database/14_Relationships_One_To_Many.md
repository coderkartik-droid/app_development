# Introduction

A one-to-many relationship means one parent record can be associated with many child records, while each child belongs to one parent. An order can have many line items.

# Why this concept exists

Storing repeated child values in one text field makes them hard to validate, search, update, and total. Separate rows preserve each child as a queryable record.

# Real-world analogy

One classroom has many students; each student record points back to its classroom.

# Internal Working

The child table stores the parent's key as a foreign key. A parent query can join or separately fetch matching children; foreign-key constraints keep references valid.

```text
orders (id PK) 1 -------- * order_items (order_id FK)
```

# Syntax

```sql
CREATE TABLE order_items (
  id INTEGER PRIMARY KEY,
  order_id INTEGER NOT NULL REFERENCES orders(id),
  product_name TEXT NOT NULL,
  quantity INTEGER NOT NULL CHECK (quantity > 0)
);
```

# Complete Example 1

This Flutter app creates one order, inserts two child items transactionally, and queries their total quantity. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openOrderDb() async {
  final file = p.join(await getDatabasesPath(), 'one_to_many_demo.db');
  return openDatabase(file, version: 1, onConfigure: (db) async {
    await db.execute('PRAGMA foreign_keys = ON');
  }, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE orders (id INTEGER PRIMARY KEY AUTOINCREMENT, label TEXT NOT NULL)',
    );
    await db.execute('''
      CREATE TABLE order_items (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        order_id INTEGER NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
        product_name TEXT NOT NULL,
        quantity INTEGER NOT NULL CHECK (quantity > 0)
      )
    ''');
  });
}

void main() => runApp(const MaterialApp(home: OrderPage()));

class OrderPage extends StatefulWidget {
  const OrderPage({super.key});
  @override
  State<OrderPage> createState() => _OrderPageState();
}

class _OrderPageState extends State<OrderPage> {
  late final Future<Database> _db = openOrderDb();
  String _summary = 'Preparing order...';

  Future<void> _createOrder() async {
    final db = await _db;
    await db.transaction((txn) async {
      final orderId = await txn.insert('orders', {'label': 'Weekly supplies'});
      await txn.insert('order_items', {
        'order_id': orderId,
        'product_name': 'Paper',
        'quantity': 3,
      });
      await txn.insert('order_items', {
        'order_id': orderId,
        'product_name': 'Pens',
        'quantity': 2,
      });
    });
    final rows = await db.rawQuery('''
      SELECT orders.label, SUM(order_items.quantity) AS item_count
      FROM orders JOIN order_items ON order_items.order_id = orders.id
      GROUP BY orders.id
    ''');
    if (mounted && rows.isNotEmpty) {
      setState(() => _summary = '${rows.first['label']}: ${rows.first['item_count']} units');
    }
  }

  @override
  void initState() {
    super.initState();
    _createOrder();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Order items')),
        body: Center(child: Text(_summary)),
      );
}
```

**Line-by-line guide:** Imports supply Flutter, path, and SQLite. `openOrderDb` enables foreign keys and creates parent and child tables. Each item references an order; cascade removes its items with the parent, and quantity must be positive. `_createOrder` inserts the parent, then two child records in one transaction. The aggregate join sums quantities per order. The state updates if the page remains mounted. `initState` runs the demonstration; `build` displays the summary. Widget tree: `MaterialApp -> OrderPage -> Scaffold -> AppBar + Center/Text`.

**Database flow:** `INSERT order -> obtain key -> INSERT many items -> JOIN/GROUP BY -> total -> UI`.

**Execution:** One order with two items totals five units. Removing the transaction can leave an order with only one item after a failure; removing the foreign key permits orphan items.

```text
+----------------------+
| Order items          |
|                      |
| Weekly supplies: 5   |
+----------------------+
```

**Common mistakes:** Saving a comma-separated item list, omitting the child foreign key index for large tables, and deleting children manually inconsistently.

**Professional use case:** Orders and order lines, projects and tasks, or categories and entries.

# Complete Example 2

Load one parent and all children:

```sql
SELECT id, product_name, quantity
FROM order_items
WHERE order_id = ?
ORDER BY id;
```

Bind the order ID. Add `CREATE INDEX idx_order_items_order_id ON order_items(order_id)` when query volume/data size justifies it. Without a filter, this returns every order's items.

# Complete Example 3

Count children per parent, including parents with none:

```sql
SELECT orders.id, orders.label, COUNT(order_items.id) AS item_count
FROM orders
LEFT JOIN order_items ON order_items.order_id = orders.id
GROUP BY orders.id;
```

`LEFT JOIN` keeps empty orders in the result. `COUNT(child.id)` yields zero for those rows; `COUNT(*)` would count the joined parent row.

# Advanced Example

Use `ON DELETE CASCADE` only when child rows have no independent meaning. Otherwise restrict deletion or archive the parent. Model quantities, timestamps, and lifecycle state explicitly for auditable order histories.

# Under the Hood

SQLite checks child foreign keys during writes when enforcement is enabled. An index on the child key helps parent-child lookups and referential checks.

# Performance Notes

Fetch children for one parent with a bounded query or join. Avoid N+1 patterns that issue one query for each list tile.

# Best Practices

- Store each child as a row.
- Enable foreign keys and index high-use child keys.
- Choose cascade/restrict behavior based on ownership.
- Wrap parent-child creation in a transaction.

# Common Mistakes

- Storing a list of IDs inside one text column.
- Forgetting foreign-key enforcement.
- Using `COUNT(*)` when counting optional child rows.
- Loading all children for every parent separately.

# Debugging

Run `PRAGMA foreign_key_check`, inspect child rows for an ID, and compare join output with expected fixtures. Test deleting a parent with and without children.

# Interview Questions

1. **Where does the foreign key go?** On the many-side child table.
2. **How represent one-to-many?** Parent primary key referenced by many child rows.
3. **Why index the child key?** To accelerate joins and parent-related operations.
4. **When use a left join?** To retain parents with no children.

# Try It Yourself

Add a query that lists every order and its item count, including empty orders.

# Challenge Project

Build a project/task database with task completion and due-date queries. Deleting a project should follow an intentional child-retention rule.

# Quick Revision Sheet

- One parent, many child records.
- Foreign key resides on the child.
- Transactions protect related inserts.
- Index child foreign keys for frequent lookups.

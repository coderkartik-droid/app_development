# Introduction

Advanced queries combine filtering, sorting, aggregation, joins, and subqueries to answer useful questions inside SQLite. Doing this work in SQL often avoids transferring and processing unnecessary records in Dart.

# Why this concept exists

Real screens need more than “load every row”: they need totals, grouped summaries, multi-table views, and filtered pages. SQL expresses these operations close to the data.

# Real-world analogy

Instead of carrying every invoice to a desk and sorting them by hand, ask the records clerk for “this month's unpaid invoices grouped by customer.”

# Internal Working

SQLite parses SQL, chooses a query plan, uses indexes where beneficial, and returns the result set. `rawQuery` completes with row maps; map results are not live views and need a new query after changes.

```text
SQL + bound args -> planner -> table/index scan + join/aggregate -> rows
```

# Syntax

```dart
final rows = await db.rawQuery(
  'SELECT category, SUM(amount_cents) AS total '
  'FROM expenses WHERE date_ms >= ? GROUP BY category ORDER BY total DESC',
  [startDateMs],
);
```

# Complete Example 1

This runnable app creates sample expenses and queries a grouped total. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openExpenseDb() async {
  final file = p.join(await getDatabasesPath(), 'advanced_query_demo.db');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE expenses (id INTEGER PRIMARY KEY, category TEXT NOT NULL, amount_cents INTEGER NOT NULL)',
    );
    await db.insert('expenses', {'id': 1, 'category': 'Food', 'amount_cents': 1200});
    await db.insert('expenses', {'id': 2, 'category': 'Travel', 'amount_cents': 800});
    await db.insert('expenses', {'id': 3, 'category': 'Food', 'amount_cents': 500});
  });
}

void main() => runApp(const MaterialApp(home: SummaryPage()));

class SummaryPage extends StatefulWidget {
  const SummaryPage({super.key});
  @override
  State<SummaryPage> createState() => _SummaryPageState();
}

class _SummaryPageState extends State<SummaryPage> {
  late final Future<Database> _db = openExpenseDb();
  List<Map<String, Object?>> _totals = [];

  Future<void> _load() async {
    final rows = await (await _db).rawQuery('''
      SELECT category, SUM(amount_cents) AS total_cents
      FROM expenses
      GROUP BY category
      ORDER BY total_cents DESC
    ''');
    if (mounted) setState(() => _totals = rows);
  }

  @override
  void initState() {
    super.initState();
    _load();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Spend by category')),
        body: ListView(
          children: _totals
              .map((row) => ListTile(
                    title: Text(row['category']! as String),
                    trailing: Text('${row['total_cents']} cents'),
                  ))
              .toList(),
        ),
      );
}
```

**Line-by-line guide:** The imports provide UI, paths, and SQLite. `openExpenseDb` creates a stable file and an expense table; sample rows are seeded only on initial creation. The state stores one database future and aggregate rows. `_load` uses `SUM` grouped by category and sorts totals descending. SQLite returns one row per category. `mounted` protects the async update. `initState` loads once. `build` shows category and total. Widget tree: `MaterialApp -> SummaryPage -> Scaffold -> AppBar + ListView/ListTile`.

**Database flow:** `SELECT -> GROUP BY category -> SUM in SQLite -> aggregate maps -> state -> list`.

**Execution:** Three sample records become two summary rows: Food 1700 and Travel 800 cents. Removing `GROUP BY` produces a single total rather than category totals; removing `SUM` loses the aggregate.

```text
+----------------------+
| Spend by category    |
|----------------------|
| Food           1700  |
| Travel          800  |
+----------------------+
```

**Common mistakes:** Grouping by a label that changes, using floating-point for money, and assuming result types without checking.

**Professional use case:** Dashboard summaries are calculated in SQL without loading the full transaction history.

# Complete Example 2

Join related tables:

```sql
SELECT expenses.id, categories.name, expenses.amount_cents
FROM expenses
JOIN categories ON categories.id = expenses.category_id
WHERE categories.name = ?;
```

The join resolves the category name from its foreign key. The placeholder safely binds the selected name. Missing join conditions can create a Cartesian product and duplicate rows.

# Complete Example 3

Keyset pagination can fetch rows after a known ID:

```sql
SELECT id, category, amount_cents
FROM expenses
WHERE id > ?
ORDER BY id
LIMIT ?;
```

Bind the last-seen ID and page size. Keyset pagination avoids the cost and instability of large offsets when rows are inserted between page requests.

# Advanced Example

Inspect a plan with `EXPLAIN QUERY PLAN SELECT ...`. If a frequent filter scans a large table, consider a matching index. Do not add indexes without measuring; plans depend on data distribution and SQLite version.

# Under the Hood

The query planner estimates available access paths. An index can satisfy filtering or ordering, and joins may use nested lookup strategies. The output is a snapshot of the query at execution time.

# Performance Notes

Project only required columns; aggregate and filter in SQL; use bounded result sets. Measure cold and warm queries on low-end target devices.

# Best Practices

- Bind values, while keeping SQL identifiers static or selected from a strict allowlist.
- Add indexes based on `EXPLAIN QUERY PLAN` and real workloads.
- Use integer minor currency units.
- Test null, empty, duplicate, and large datasets.

# Common Mistakes

- Building SQL from untrusted input.
- Selecting `*` for a narrow screen.
- Omitting a join predicate.
- Using `OFFSET` for deep, frequently changing lists without considering alternatives.

# Debugging

Run the query with known fixture data, inspect its result shape, and check the plan. Confirm that aggregate aliases and expected numeric types match mapping code.

# Interview Questions

1. **What does `GROUP BY` do?** Produces groups for aggregate calculations.
2. **Why use a join?** To combine related rows through a relationship key.
3. **What is a query plan?** The engine's chosen strategy for executing a query.
4. **Why keyset pagination?** It avoids deep offset scans and is more stable for changing data.

# Try It Yourself

Add a monthly filter and compute average expense per category. Test an empty month.

# Challenge Project

Build an expense report screen with category totals, date filters, and bounded transaction pages; add indexes only after examining query plans.

# Quick Revision Sheet

- Filter, join, group, aggregate, sort, and limit in SQL.
- Bind values; do not concatenate user input.
- `EXPLAIN QUERY PLAN` reveals scans and index use.
- Aggregates keep large raw sets out of Dart memory.

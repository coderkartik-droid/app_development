# Introduction

Database performance optimization means making the queries and writes needed by the app complete with acceptable latency and resource use, while preserving correctness. Measure first; optimize the actual workload.

# Why this concept exists

Large tables, repeated queries, unnecessary columns, and missing indexes can make screens slow and consume memory or battery. Thoughtful query design keeps interactions responsive.

# Real-world analogy

An index is a catalog in a warehouse: it speeds up finding a matching box, but maintaining the catalog takes extra work whenever boxes move.

# Internal Working

SQLite plans each query and chooses scans or indexes. Dart awaits the result, maps it, and updates state. Performance includes database execution, bridge overhead, decoding, allocation, and UI rendering—not just SQL time.

```text
query -> planner -> storage reads -> result transfer -> Dart mapping -> frame
```

# Syntax

```sql
CREATE INDEX idx_expenses_date ON expenses(date_ms);
SELECT id, amount_cents FROM expenses WHERE date_ms >= ? ORDER BY date_ms LIMIT ?;
```

# Complete Example 1

The app demonstrates bounded reads and an index on the date column. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openIndexedDb() async {
  final file = p.join(await getDatabasesPath(), 'performance_demo.db');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE expenses (id INTEGER PRIMARY KEY, date_ms INTEGER NOT NULL, amount_cents INTEGER NOT NULL)',
    );
    await db.execute(
      'CREATE INDEX idx_expenses_date ON expenses(date_ms)',
    );
  });
}

void main() => runApp(const MaterialApp(home: RecentExpensesPage()));

class RecentExpensesPage extends StatefulWidget {
  const RecentExpensesPage({super.key});
  @override
  State<RecentExpensesPage> createState() => _RecentExpensesPageState();
}

class _RecentExpensesPageState extends State<RecentExpensesPage> {
  late final Future<Database> _db = openIndexedDb();
  List<Map<String, Object?>> _rows = [];

  Future<void> _load() async {
    final rows = await (await _db).query(
      'expenses',
      columns: ['id', 'amount_cents'],
      where: 'date_ms >= ?',
      whereArgs: [DateTime.now().subtract(const Duration(days: 30)).millisecondsSinceEpoch],
      orderBy: 'date_ms DESC',
      limit: 50,
    );
    if (mounted) setState(() => _rows = rows);
  }

  @override
  void initState() {
    super.initState();
    _load();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Recent expenses')),
        body: ListView(
          children: _rows
              .map((row) => ListTile(
                    title: Text('Expense ${row['id']}'),
                    trailing: Text('${row['amount_cents']} cents'),
                  ))
              .toList(),
        ),
      );
}
```

**Line-by-line guide:** Imports provide UI, path handling, and SQLite. `openIndexedDb` creates a table and date index. The page caches its database future and query results. `_load` selects only two columns, applies a recent-date predicate with a bound value, sorts descending, and caps output at 50. State updates only if mounted. `initState` starts the query; `build` displays bounded results. Widget tree: `MaterialApp -> RecentExpensesPage -> Scaffold -> ListView/ListTile`.

**Database flow:** `date index lookup -> at most 50 projected rows -> Dart maps -> state -> visible list`.

**Execution:** The database applies the date condition and limit before transferring data. Removing `limit` can return an unbounded history; removing the index may cause a table scan for large datasets. An index still needs measurement because tiny tables may not benefit.

```text
+----------------------+
| Recent expenses      |
|----------------------|
| Expense 42      900  |
| Expense 41      250  |
+----------------------+
```

**Common mistakes:** Indexing every column, optimizing without measuring, and assuming a faster query fixes expensive widget construction.

**Professional use case:** A transaction history with date filtering, stable paging, and a small on-screen window.

# Complete Example 2

Inspect a query plan:

```dart
final plan = await db.rawQuery(
  'EXPLAIN QUERY PLAN SELECT id FROM expenses WHERE date_ms >= ?',
  [cutoffMs],
);
```

Look for whether SQLite uses a suitable index or scans the table. Plan text can vary by SQLite version, so tests should avoid brittle string equality.

# Complete Example 3

Fetch only a page:

```sql
SELECT id, date_ms, amount_cents
FROM expenses
WHERE id > ?
ORDER BY id
LIMIT ?;
```

Stable keyset pagination avoids loading the entire table and can outperform deep offsets. Define sorting and duplicate-timestamp behavior explicitly.

# Advanced Example

Measure end-to-end latency with realistic datasets and a release-like build. Separate time spent opening, querying, mapping, and rendering. For bulk writes, use a transaction; for read-heavy access, evaluate covering/composite indexes against the exact predicates and sort order.

# Under the Hood

Indexes are B-tree structures and increase database size. The query planner uses schema statistics and available constraints to choose an access path. Returning rows still incurs conversion and Dart allocation.

# Performance Notes

- Limit and paginate large tables.
- Project required columns only.
- Index columns used in common filters/joins/orderings after measurement.
- Batch related writes transactionally.
- Avoid repeated per-item queries.

# Best Practices

- Capture a baseline before optimization.
- Use representative data volume and device hardware.
- Recheck query plans after schema changes.
- Keep correctness tests while changing indexes or query shapes.

# Common Mistakes

- Adding indexes without considering write cost.
- Using `SELECT *` in every screen.
- Loading all records into memory to filter in Dart.
- Treating database duration as the full screen latency.

# Debugging

Record operation duration in development, inspect `EXPLAIN QUERY PLAN`, and profile widget builds separately. Check for accidental duplicate queries in lifecycle methods.

# Interview Questions

1. **What does an index trade?** Storage and write overhead for faster matching reads.
2. **Why limit results?** Controls query work, transfer, and memory.
3. **How investigate a slow query?** Measure it, inspect the plan, and validate indexes/data distribution.
4. **What is N+1 querying?** One query for a list plus a separate query for each item.

# Try It Yourself

Benchmark a filter with and without an index at increasing row counts. Record query and rendering time separately.

# Challenge Project

Optimize a 50,000-row transaction history for date filtering and paging. Demonstrate plan changes and validate correctness.

# Quick Revision Sheet

- Measure first; optimize end-to-end.
- Filter, project, and limit in SQL.
- Indexes accelerate selected access paths but cost writes/storage.
- Inspect query plans and avoid N+1 patterns.

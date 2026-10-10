# Introduction

This mini project builds the database core of an offline expense tracker. It stores expense descriptions, categories, integer amounts in minor currency units, and timestamps.

# Why this concept exists

Financial records need durable storage, accurate arithmetic, useful filters, and summaries. A relational database makes category links and aggregate queries explicit.

# Real-world analogy

Each expense is a dated receipt filed under a category; monthly reports total the receipts without changing the original records.

# Internal Working

The form creates a domain input; the repository inserts it. SQLite enforces required values and provides grouped totals. After the write, the screen reloads or watches the database query.

```text
Expense form -> repository -> INSERT expense -> SQLite
             <- summary query <- GROUP BY category <- records
```

# Syntax

```sql
SELECT category, SUM(amount_cents) AS total_cents
FROM expenses
WHERE spent_at_ms >= ? AND spent_at_ms < ?
GROUP BY category
ORDER BY total_cents DESC;
```

# Complete Example 1

This runnable example creates the expense table, inserts a sample expense, computes the total in SQLite, and displays it. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openExpenseTrackerDb() async {
  final file = p.join(await getDatabasesPath(), 'expense_tracker.db');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute('''
      CREATE TABLE expenses (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        category TEXT NOT NULL,
        amount_cents INTEGER NOT NULL CHECK (amount_cents > 0),
        spent_at_ms INTEGER NOT NULL
      )
    ''');
  });
}

void main() => runApp(const MaterialApp(home: ExpensePage()));

class ExpensePage extends StatefulWidget {
  const ExpensePage({super.key});
  @override
  State<ExpensePage> createState() => _ExpensePageState();
}

class _ExpensePageState extends State<ExpensePage> {
  late final Future<Database> _db = openExpenseTrackerDb();
  int _totalCents = 0;

  Future<void> _refresh() async {
    final result = await (await _db).rawQuery(
      'SELECT COALESCE(SUM(amount_cents), 0) AS total FROM expenses',
    );
    if (mounted) setState(() => _totalCents = result.single['total']! as int);
  }

  Future<void> _addSample() async {
    await (await _db).insert('expenses', {
      'title': 'Lunch',
      'category': 'Food',
      'amount_cents': 1250,
      'spent_at_ms': DateTime.now().millisecondsSinceEpoch,
    });
    await _refresh();
  }

  @override
  void initState() {
    super.initState();
    _refresh();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Expense tracker')),
        body: Center(child: Text('Total: $_totalCents cents')),
        floatingActionButton: FloatingActionButton(
          onPressed: _addSample,
          child: const Icon(Icons.add),
        ),
      );
}
```

**Line-by-line guide:** Imports load Flutter, path, and SQLite. `openExpenseTrackerDb` creates a stable database and an expenses table. The schema requires all fields and rejects non-positive amounts. The state owns one database future and total. `_refresh` uses `COALESCE` so an empty table reports zero instead of null. `_addSample` stores 1250 minor units and the current timestamp, then reloads the summary. `initState` loads existing data. The FAB inserts; the centered text displays the total. Widget tree: `MaterialApp -> ExpensePage -> Scaffold -> AppBar + Center/Text + FAB`.

**Database flow:** `insert cents/timestamp -> SUM(amount_cents) -> state -> total label`.

**Execution:** Initially total is zero. Each press adds 1250 cents. If `COALESCE` is removed, an empty table produces null; if amount were stored as floating-point major units, rounding could introduce inaccuracies.

```text
+----------------------+
| Expense tracker      |
|                      |
| Total: 1250 cents    |
|                 [+]  |
+----------------------+
```

**Common mistakes:** Using binary floating point for money, storing a formatted currency string, and assuming timestamps use the same timezone/units.

**Professional use case:** Offline expense entry and summaries with transparent integer arithmetic.

# Complete Example 2

List a date range with bound start/end values and a limit. Use a half-open interval (`>= start AND < end`) so adjacent days do not overlap. Convert local calendar boundaries carefully; timestamp storage should use one documented timezone convention.

# Complete Example 3

Normalize categories into a `categories` table and store `category_id` in each expense. Use a foreign key and join for display. Migrate existing category text values instead of discarding them.

# Advanced Example

Add monthly/category indexes based on actual report queries; wrap expense and recurring-rule writes in transactions when they form one logical action. Export/backup and retention choices must be explicit for user financial records.

# Under the Hood

SQLite stores cents and milliseconds as integers. Aggregate queries run in the engine. Flutter state changes only after the query result is mapped and applied.

# Performance Notes

Use date filters, projections, and pagination for history. Keep summaries in SQL and add indexes after measuring query plans.

# Best Practices

- Store money as integer minor units plus currency code where multi-currency applies.
- Use bound parameters and constraints.
- Version schema changes and retain financial history.
- Test date boundaries, empty months, and large values.

# Common Mistakes

- Floating-point money arithmetic.
- Deleting old entries when a category changes.
- Assuming device local time is a durable universal timestamp.
- Running aggregate queries repeatedly in every list tile.

# Debugging

Verify stored cents, timestamp units, date range boundaries, and `SUM` result types. Seed a known fixture and compare expected totals.

# Interview Questions

1. **How store currency amounts accurately?** Integer minor units and explicit currency metadata.
2. **Why half-open date intervals?** They avoid overlap between adjacent periods.
3. **Where should totals be calculated?** Often in SQL for large histories.
4. **What should happen when a category is removed?** Follow an explicit retention/reassignment policy.

# Try It Yourself

Add a category summary query and a date-range filter. Test at midnight boundaries and for a month with no entries.

# Challenge Project

Build an expense tracker with add/edit/delete, categories, date filtering, monthly reports, database migrations, and repository tests. Keep all financial calculations in integer minor units.

# Quick Revision Sheet

- Expenses are durable rows with constraints.
- Store money in integer minor units.
- Filter date ranges with explicit timestamp semantics.
- Aggregate in SQL; migrate data rather than dropping it.

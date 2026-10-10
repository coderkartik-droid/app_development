# Introduction

This chapter reviews database topics commonly discussed in Flutter interviews: relational modeling, SQL operations, transactions, indexes, migrations, and the boundary between persistence and UI.

# Why this concept exists

Interviewers want to know whether you can preserve correctness and user data under real lifecycle changes—not merely recall SQL keywords.

# Real-world analogy

An interview answer is a design review: explain the rule, show the structure, then describe failure and trade-offs.

# Internal Working

A strong answer follows the path from Dart operation to database constraint/result, then explains how the app state reflects it.

```text
requirement -> model/schema -> transaction/query -> typed result -> UI/test
```

# Syntax

```sql
CREATE INDEX idx_items_category ON items(category_id);
```

An index can speed up category filtering, but consumes storage and adds write work.

# Complete Example 1

A small runnable Flutter example demonstrates a parameterized query. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openInterviewDb() async {
  final file = p.join(await getDatabasesPath(), 'interview_demo.db');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE products (id INTEGER PRIMARY KEY, name TEXT NOT NULL)',
    );
    await db.insert('products', {'id': 1, 'name': 'Keyboard'});
  });
}

void main() => runApp(const MaterialApp(home: InterviewPage()));

class InterviewPage extends StatefulWidget {
  const InterviewPage({super.key});
  @override
  State<InterviewPage> createState() => _InterviewPageState();
}

class _InterviewPageState extends State<InterviewPage> {
  late final Future<Database> _db = openInterviewDb();
  String _name = 'Loading';

  Future<void> _find() async {
    final rows = await (await _db).query(
      'products',
      columns: ['name'],
      where: 'id = ?',
      whereArgs: [1],
      limit: 1,
    );
    if (mounted) {
      setState(() => _name = rows.isEmpty ? 'Not found' : rows.single['name']! as String);
    }
  }

  @override
  void initState() {
    super.initState();
    _find();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Interview query')),
        body: Center(child: Text(_name)),
      );
}
```

**Line-by-line guide:** Imports load UI, path, and SQLite. `openInterviewDb` creates a stable file, product table, and one fixture row on first creation. The page owns a database future and displayed name. `_find` selects only the required column, binds the ID, limits the result, handles no match, and updates state safely. `initState` runs it once. Widget tree: `MaterialApp -> InterviewPage -> Scaffold -> AppBar + Center/Text`.

**Database flow:** `id parameter -> indexed primary-key lookup -> result or empty -> state -> text`.

**Execution:** The sample displays Keyboard. Without the `where` condition the query could return another product; without the empty check, `single` throws when no record matches.

```text
+----------------------+
| Interview query      |
|                      |
| Keyboard             |
+----------------------+
```

**Common mistakes:** Giving definitions without explaining trade-offs, constraints, or failure behavior.

**Professional use case:** Explain a production decision in terms of workload, data integrity, and migration safety.

# Complete Example 2

**Q: How do you model many-to-many?** Use a junction table with foreign keys to both parent tables and a composite unique/primary key to prevent duplicate pairs. Add relationship attributes to the junction row.

# Complete Example 3

**Q: How do you prevent partial multi-table writes?** Execute them inside one transaction, use its transaction handle, and allow failures to propagate so rollback occurs.

# Advanced Example

Practice trade-off questions:

- Indexing: faster selected reads, more storage and write work.
- Normalization: less duplication and better update integrity, sometimes more joins.
- Repository: testable data boundary, but avoid meaningless abstraction.
- Local database: durable on-device data, not cross-device synchronization by itself.

# Under the Hood

SQLite parses SQL, checks constraints, chooses a query plan, and reads/writes file pages. Flutter's async boundary returns results; application code maps them and updates state.

# Performance Notes

Discuss measured query plans and end-to-end latency, not guesses. Large lists need bounded queries and often stable pagination.

# Best Practices

- Answer with a concrete schema/query.
- Mention error, empty, and concurrency cases.
- Describe migration and test strategy.
- State trade-offs rather than claiming a tool is always fastest.

# Common Mistakes

- Confusing primary and foreign keys.
- Saying indexes make every operation faster.
- Claiming `onCreate` changes existing databases.
- Ignoring data-loss risk in migration answers.

# Debugging

Talk through path, schema, query plan, arguments, row counts, and UI refresh. Explain how a temporary database test reproduces the issue.

# Interview Questions

1. **Primary key vs foreign key?** A primary key identifies a row; a foreign key references another table's key.
2. **What is normalization?** Organizing related data to reduce duplication and update anomalies.
3. **What does ACID mean?** Atomicity, consistency, isolation, durability.
4. **When use an index?** For measured frequent predicates/orderings/joins that benefit from it.
5. **How preserve data on schema change?** Versioned, ordered, tested migrations.
6. **What does a zero update count mean?** No row matched the update predicate.
7. **Why parameterize SQL?** Safe value binding and correct escaping.
8. **How test a repository?** Unit tests for mapping/contracts and integration tests against a temporary database.

# Try It Yourself

Answer each question aloud using a schema diagram, one SQL snippet, and one failure case.

# Challenge Project

Design an expense schema on paper, explain its indexes and migration plan, then implement and test a transaction that creates an expense and category summary record.

# Quick Revision Sheet

- Keys identify and relate records.
- Transactions prevent partial logical changes.
- Indexes trade write/storage cost for selected read speed.
- Migrations preserve existing data.
- Explain assumptions, failure paths, tests, and trade-offs.

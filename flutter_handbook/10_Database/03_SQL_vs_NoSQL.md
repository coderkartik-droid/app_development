# Introduction

SQL databases organize information into tables with declared columns and relationships. NoSQL is a broad family of non-relational models, such as document, key-value, and graph databases. The choice depends on access patterns and consistency needs—not on which label sounds newer.

# Why this concept exists

Applications model data in different shapes. A relational model is strong when records connect and need joins or constraints; document-oriented models can be convenient when data is naturally grouped and accessed as a whole.

# Real-world analogy

SQL resembles a set of linked spreadsheets with enforced column rules. A document database resembles folders of self-contained forms, each capable of carrying nested sections.

# Internal Working

In a relational engine, SQL describes set-based operations over tables; keys and constraints define relationships. A document engine stores nested records and commonly retrieves a document as a unit. Flutter packages map either engine's results into Dart objects.

```text
Flutter repository -> database-specific adapter -> SQL tables / documents
                   <- rows, joined data, or document objects
```

# Syntax

```sql
-- Relational query
SELECT id, name FROM products WHERE price_cents < ?;
```

```dart
// Document-style conceptual access; exact API depends on the database.
final product = await collection.get(productId);
```

# Complete Example 1

This runnable Flutter example demonstrates a relational SQLite model with `sqflite`. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openCatalog() async {
  final file = p.join(await getDatabasesPath(), 'sql_nosql_demo.db');
  return openDatabase(file, version: 1, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE products (id INTEGER PRIMARY KEY, name TEXT NOT NULL, price_cents INTEGER NOT NULL)',
    );
    await db.insert('products', {
      'id': 1,
      'name': 'Notebook',
      'price_cents': 450,
    });
  });
}

void main() => runApp(const MaterialApp(home: CatalogPage()));

class CatalogPage extends StatefulWidget {
  const CatalogPage({super.key});
  @override
  State<CatalogPage> createState() => _CatalogPageState();
}

class _CatalogPageState extends State<CatalogPage> {
  late final Future<Database> _db = openCatalog();
  List<Map<String, Object?>> _products = [];

  Future<void> _load() async {
    final rows = await (await _db).query(
      'products',
      columns: ['name', 'price_cents'],
      where: 'price_cents < ?',
      whereArgs: [1000],
    );
    if (mounted) setState(() => _products = rows);
  }

  @override
  void initState() {
    super.initState();
    _load();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Relational catalog')),
        body: ListView(
          children: _products
              .map((row) => ListTile(
                    title: Text(row['name']! as String),
                    subtitle: Text('${row['price_cents']} cents'),
                  ))
              .toList(),
        ),
      );
}
```

**Line-by-line guide:** The imports enable UI, filesystem path joining, and SQLite. `openCatalog` opens one local file and creates a table on first use. Its primary key identifies a product; required fields prevent incomplete products. A sample row is inserted during first creation only. The page owns the open future and product rows. `_load` projects two columns and filters prices using a bound parameter. `mounted` prevents a late async response from updating disposed state. `build` maps each row to a list tile. The widget tree is `MaterialApp -> CatalogPage -> Scaffold -> AppBar + ListView/ListTile`.

**Database flow:** `query products where price < 1000 -> SQLite filters rows -> Dart maps results -> ListView`.

**Execution:** On first run, the catalog is created and seeded; subsequent runs reuse the file. Removing the `where` returns all products, while removing parameter binding and interpolating input risks malformed SQL or injection.

```text
+----------------------+
| Relational catalog   |
|----------------------|
| Notebook             |
| 450 cents            |
+----------------------+
```

**Common mistakes:** Comparing unlike models only by speed, storing money in floating-point without considering rounding, and assuming every NoSQL database has the same query behavior.

**Professional use case:** Relational product/order data with referential constraints and multi-table reporting.

# Complete Example 2

A document-shaped value might be represented in Dart as:

```dart
final productDocument = {
  'id': 'p1',
  'name': 'Notebook',
  'priceCents': 450,
  'tags': ['paper', 'stationery'],
};
```

This is a Dart map illustrating a document's shape, not persistent storage by itself. A document database adapter would serialize and store it. The nested list is convenient when tags are read with the product; duplicating data into documents complicates global updates.

# Complete Example 3

A relational join expresses shared data without copying it:

```sql
SELECT products.name, categories.name
FROM products
JOIN categories ON categories.id = products.category_id;
```

The foreign key connects each product to a category. Without the join, the category name must either be queried separately or duplicated, each with different consistency trade-offs.

# Advanced Example

Choose the model from the dominant read/write paths. Normalize relational facts that change independently; consider embedding document data that is always read and updated together. For either model, define transaction boundaries, backup strategy, schema evolution, and data ownership before shipping.

# Under the Hood

SQL engines use query planners, indexes, constraints, and transactions. Document engines commonly index selected fields and work with structured records. Both can be durable and both can be poorly designed; “NoSQL” does not imply “no schema.”

# Performance Notes

Benchmark representative data and queries. A relational join can outperform repeated lookups; a document read can avoid joins for a known aggregate. Indexes, serialization, and write amplification affect both.

# Best Practices

- Start from required queries and invariants.
- Prefer relational tables for connected records and strong constraints.
- Use document storage where aggregate access is naturally document-oriented.
- Keep persistence behind an abstraction without hiding important consistency behavior.

# Common Mistakes

- Treating SQL/NoSQL as a universal winner-takes-all choice.
- Ignoring migration and backup requirements.
- Duplicating mutable facts without a consistency plan.
- Assuming a Dart map is already persisted.

# Debugging

Inspect actual stored data and indexes. Test missing fields, duplicate IDs, concurrent writes, and migrations. Separate adapter errors from engine errors.

# Interview Questions

1. **What is a relational database?** A system that stores structured records in related tables.
2. **Does NoSQL mean no schema?** No; data shape and validation still exist, possibly enforced differently.
3. **What is normalization?** Structuring relational data to reduce unnecessary duplication and anomalies.
4. **How should teams choose?** Based on access patterns, consistency, relationships, scale, and operational needs.

# Try It Yourself

Model products and categories both as normalized tables and as a nested document. List the changes needed when a category name changes.

# Challenge Project

Design an offline product catalog. Write down five expected queries, then justify SQL or a document database before implementation.

# Quick Revision Sheet

- SQL: relational tables, queries, constraints, joins.
- NoSQL: multiple non-relational models, often document/key-value/graph.
- Neither guarantees good design or automatic speed.
- Select based on access patterns and integrity requirements.
- Dart objects need a database adapter to become durable.

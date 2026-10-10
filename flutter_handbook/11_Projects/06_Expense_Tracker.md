# Project Overview

Create a small expense tracker that records an item, category, and amount. This runnable version keeps records in memory and stores amounts as integer cents to avoid floating-point currency arithmetic.

# Features

- Add an expense with validation
- Choose a simple category
- Show a list of entries
- Calculate the total in integer minor units
- Delete an entry

# Folder Structure

```text
lib/
  main.dart
  models/expense.dart
  data/expense_repository.dart
  screens/expense_page.dart
  widgets/expense_form.dart
```

# ASCII UI Preview

```text
+--------------------------+
| Expenses                 |
| Total: $24.50            |
| [Lunch              $12] |
| [Bus fare           $3]  |
| [Notebook           $9.5]|
|                    [+]   |
+--------------------------+
```

# Step-by-Step Development Plan

1. Define an immutable expense model.
2. Parse a decimal amount into integer cents.
3. Build a validated entry dialog.
4. Derive the total from the expense list.
5. Add durable storage and date/category reports after the basic flow works.

# Complete Runnable Code

Paste this into `lib/main.dart` in a Flutter project; it requires no package. Values disappear after app restart because this version is intentionally in-memory.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const ExpenseApp());

class ExpenseApp extends StatelessWidget {
  const ExpenseApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        title: 'Expenses',
        theme: ThemeData(colorSchemeSeed: Colors.green, useMaterial3: true),
        home: const ExpensePage(),
      );
}

class Expense {
  const Expense({
    required this.id,
    required this.title,
    required this.category,
    required this.amountCents,
  });
  final int id;
  final String title;
  final String category;
  final int amountCents;
}

class ExpensePage extends StatefulWidget {
  const ExpensePage({super.key});

  @override
  State<ExpensePage> createState() => _ExpensePageState();
}

class _ExpensePageState extends State<ExpensePage> {
  final List<Expense> _expenses = [];
  int _nextId = 1;

  int get _totalCents =>
      _expenses.fold(0, (total, expense) => total + expense.amountCents);

  Future<void> _addExpense() async {
    final titleController = TextEditingController();
    final amountController = TextEditingController();
    final values = await showDialog<(String, int)>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Add expense'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(
              controller: titleController,
              decoration: const InputDecoration(labelText: 'Description'),
            ),
            TextField(
              controller: amountController,
              keyboardType: const TextInputType.numberWithOptions(decimal: true),
              decoration: const InputDecoration(labelText: 'Amount'),
            ),
          ],
        ),
        actions: [
          TextButton(onPressed: () => Navigator.pop(context), child: const Text('Cancel')),
          FilledButton(
            onPressed: () {
              final title = titleController.text.trim();
              final amount = double.tryParse(amountController.text.trim());
              if (title.isNotEmpty && amount != null && amount > 0) {
                Navigator.pop(context, (title, (amount * 100).round()));
              }
            },
            child: const Text('Save'),
          ),
        ],
      ),
    );
    titleController.dispose();
    amountController.dispose();
    if (values == null) return;
    setState(() {
      _expenses.insert(
        0,
        Expense(
          id: _nextId++,
          title: values.$1,
          category: 'General',
          amountCents: values.$2,
        ),
      );
    });
  }

  void _remove(int id) {
    setState(() => _expenses.removeWhere((expense) => expense.id == id));
  }

  String _money(int cents) =>
      '\$${(cents ~/ 100)}.${(cents % 100).toString().padLeft(2, '0')}';

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(
          title: const Text('Expenses'),
          bottom: PreferredSize(
            preferredSize: const Size.fromHeight(44),
            child: Padding(
              padding: const EdgeInsets.only(bottom: 12),
              child: Text('Total: ${_money(_totalCents)}'),
            ),
          ),
        ),
        body: _expenses.isEmpty
            ? const Center(child: Text('No expenses recorded.'))
            : ListView.builder(
                itemCount: _expenses.length,
                itemBuilder: (context, index) {
                  final expense = _expenses[index];
                  return ListTile(
                    key: ValueKey(expense.id),
                    title: Text(expense.title),
                    subtitle: Text(expense.category),
                    trailing: Row(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        Text(_money(expense.amountCents)),
                        IconButton(
                          tooltip: 'Delete expense',
                          onPressed: () => _remove(expense.id),
                          icon: const Icon(Icons.delete_outline),
                        ),
                      ],
                    ),
                  );
                },
              ),
        floatingActionButton: FloatingActionButton(
          onPressed: _addExpense,
          child: const Icon(Icons.add),
        ),
      );
}
```

# Line-by-Line Explanation

- `Expense` is immutable and stores the monetary value as `amountCents`.
- `_totalCents` uses `fold` to derive the total from source records, avoiding a second total that could drift out of sync.
- `_addExpense` opens a dialog and gathers description and amount.
- `double.tryParse` validates syntax; the app rejects blank titles, non-numeric values, and non-positive amounts.
- The parsed amount is converted to cents with rounding. For strict financial input, parse decimal digits directly rather than relying on binary floating point.
- Controllers are disposed after the dialog closes.
- `_money` formats integer cents as dollars and two digits.
- `_remove` deletes by stable ID and `setState` refreshes the total and list.
- The app bar's `PreferredSize` displays the total; the body switches between empty and populated states.

Removing integer-cent storage invites rounding errors. Removing the positive-value validation can allow nonsensical entries. This example's decimal parse is suitable only as a simple tutorial; robust currency parsing should avoid floating-point conversion.

# Widget Tree

```text
ExpenseApp > MaterialApp > ExpensePage > Scaffold
├── AppBar + total summary
├── Empty-state text OR ListView.builder > ListTile
└── FAB > AlertDialog > description/amount fields + actions
```

# State Management Used

The one-screen sample uses local state and derives totals from the list. For a larger app, place operations and aggregation behind an expense repository and expose state consistently.

# API/Database Used (if applicable)

None. The data is volatile in memory. A real tracker should persist records locally with constraints, migrations, and tests.

# Common Mistakes

- Storing currency as `double` and summing it directly.
- Displaying a total that is separately mutated from the entries.
- Accepting zero/negative values unintentionally.
- Calling this tracker persistent when it is memory-only.

# Improvements

- Parse decimal strings into cents without floating point.
- Add categories, dates, budgets, and date-range summaries.
- Store data in SQLite and test migration preservation.
- Add currency selection and localized formatting.

# Challenge Version

Build a monthly report with category totals, edit/delete confirmation, and a database-backed repository. Define how refunds and multiple currencies should be represented.

# Summary

This project combines input validation, immutable records, deletion by ID, and derived totals. Its explicit integer-cents model introduces an important financial-data rule.

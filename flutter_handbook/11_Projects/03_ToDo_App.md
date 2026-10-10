# Project Overview

Build a small to-do list with add, complete, and delete actions. Tasks live in memory for this beginner version; the design can later be moved behind a repository and persisted in a database.

# Features

- Add non-empty tasks
- Mark tasks complete/incomplete
- Delete an individual task
- Empty-state guidance
- Stable per-task model and UI identity

# Folder Structure

```text
lib/
  main.dart
  models/task.dart
  widgets/task_tile.dart
  data/task_repository.dart
```

# ASCII UI Preview

```text
+--------------------------+
| My tasks                 |
| [Write a Flutter app] [+]|
|--------------------------|
| [ ] Read chapter     [x] |
| [✓] Buy groceries   [x]  |
+--------------------------+
```

# Step-by-Step Development Plan

1. Define a task model with an ID, title, and completion flag.
2. Store tasks in state and render them with `ListView`.
3. Validate and clear the add field.
4. Implement completion and delete by ID.
5. Add persistence only after the in-memory behavior is clear.

# Complete Runnable Code

Use this single-file implementation in `lib/main.dart`. It needs no external dependencies.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const TodoApp());

class TodoApp extends StatelessWidget {
  const TodoApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        title: 'To-do',
        theme: ThemeData(colorSchemeSeed: Colors.teal, useMaterial3: true),
        home: const TodoPage(),
      );
}

class TodoItem {
  TodoItem({required this.id, required this.title, this.isDone = false});
  final int id;
  final String title;
  bool isDone;
}

class TodoPage extends StatefulWidget {
  const TodoPage({super.key});

  @override
  State<TodoPage> createState() => _TodoPageState();
}

class _TodoPageState extends State<TodoPage> {
  final _controller = TextEditingController();
  final List<TodoItem> _items = [];
  int _nextId = 0;

  void _addTask() {
    final title = _controller.text.trim();
    if (title.isEmpty) return;
    setState(() {
      _items.insert(0, TodoItem(id: _nextId++, title: title));
      _controller.clear();
    });
  }

  void _toggleTask(TodoItem item, bool? value) {
    setState(() => item.isDone = value ?? false);
  }

  void _deleteTask(int id) {
    setState(() => _items.removeWhere((item) => item.id == id));
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('My tasks')),
        body: Column(
          children: [
            Padding(
              padding: const EdgeInsets.all(16),
              child: Row(
                children: [
                  Expanded(
                    child: TextField(
                      controller: _controller,
                      onSubmitted: (_) => _addTask(),
                      decoration: const InputDecoration(
                        labelText: 'New task',
                        border: OutlineInputBorder(),
                      ),
                    ),
                  ),
                  IconButton(
                    tooltip: 'Add task',
                    onPressed: _addTask,
                    icon: const Icon(Icons.add_circle),
                  ),
                ],
              ),
            ),
            Expanded(
              child: _items.isEmpty
                  ? const Center(child: Text('No tasks yet. Add one above.'))
                  : ListView.builder(
                      itemCount: _items.length,
                      itemBuilder: (context, index) {
                        final item = _items[index];
                        return CheckboxListTile(
                          key: ValueKey(item.id),
                          value: item.isDone,
                          title: Text(
                            item.title,
                            style: item.isDone
                                ? const TextStyle(
                                    decoration: TextDecoration.lineThrough,
                                  )
                                : null,
                          ),
                          onChanged: (value) => _toggleTask(item, value),
                          secondary: IconButton(
                            tooltip: 'Delete task',
                            onPressed: () => _deleteTask(item.id),
                            icon: const Icon(Icons.delete_outline),
                          ),
                        );
                      },
                    ),
            ),
          ],
        ),
      );
}
```

# Line-by-Line Explanation

- `TodoItem` holds one task and its completion state; IDs remain stable if the list order changes.
- `_controller` manages text input and `_items` is the in-memory collection.
- `_addTask` trims whitespace, ignores empty input, inserts at the top, and clears the field within `setState`.
- `_toggleTask` changes completion; `value ?? false` safely handles the nullable checkbox callback.
- `_deleteTask` removes only the item with the matching ID.
- `dispose` releases the controller.
- The entry `Row` uses `Expanded` so the field uses available width while the icon remains visible.
- `Expanded` gives the list the remaining vertical space in the `Column`.
- The empty state is shown when the list has no elements; otherwise `ListView.builder` creates visible rows lazily.
- `ValueKey(item.id)` helps Flutter preserve the right row state during list changes.
- `CheckboxListTile` combines a checkbox, title, and callback; the delete control has a tooltip.

Removing `Expanded` around the list can cause a vertical layout error. Removing the `setState` calls changes the Dart objects but does not schedule a UI rebuild.

# Widget Tree

```text
TodoApp > MaterialApp > TodoPage > Scaffold
├── AppBar
└── Column
    ├── Padding > Row > Expanded(TextField) + add button
    └── Expanded
        ├── Empty-state text, or
        └── ListView.builder > CheckboxListTile + delete action
```

# State Management Used

Local `StatefulWidget` state stores the task list. For multiple screens, filtering, or persistence, move operations into a repository and expose state through the app's chosen state-management approach.

# API/Database Used (if applicable)

None. This example loses tasks when the process restarts. Persistence is intentionally left for the database/local-storage chapters.

# Common Mistakes

- Accepting whitespace-only tasks.
- Using list index as identity when rows can be reordered.
- Forgetting the empty state.
- Assuming in-memory state persists after restart.

# Improvements

- Add due dates, filters, and undo for deletion.
- Add a database-backed repository.
- Add tests for add, toggle, and delete behavior.
- Confirm before destructive actions when tasks are important.

# Challenge Version

Add task editing and priority sorting while keeping identity stable. Then persist tasks through a versioned local database and verify they survive restart.

# Summary

This project practices model-based UI, form validation, stable keys, and local state. It forms a useful starting point for a database-backed productivity app.

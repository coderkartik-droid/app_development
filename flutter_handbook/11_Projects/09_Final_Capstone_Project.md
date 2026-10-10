# Project Overview

The capstone is a personal learning planner: a local-first prototype for tracking learning goals and the next study actions. It combines a dashboard, typed models, form validation, derived progress, and a clear upgrade path to database persistence.

# Features

- Three sample learning goals
- Add a new goal
- Mark a goal complete
- Derive completed-goal progress
- Empty-input validation and responsive list layout

# Folder Structure

```text
lib/
  main.dart
  models/learning_goal.dart
  data/goal_repository.dart
  screens/dashboard_page.dart
  widgets/goal_card.dart
test/
  goal_repository_test.dart
```

# ASCII UI Preview

```text
+--------------------------+
| Learning Planner         |
| Progress: 1 of 3         |
| [██████------]            |
|--------------------------|
| Flutter fundamentals [✓] |
| Database design      [ ] |
| Build a project      [ ] |
|                    [+]   |
+--------------------------+
```

# Step-by-Step Development Plan

1. Define a goal model with identity, title, and completion state.
2. Seed a small sample list and derive progress from it.
3. Add validated goal creation.
4. Toggle completion by stable ID.
5. Extract a repository and persist records once the UI flow is stable.
6. Add tests for validation, completion, and progress.

# Complete Runnable Code

Use this in `lib/main.dart` in a standard Flutter project. No external package is required. Data is in memory for this capstone's runnable baseline.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const LearningPlannerApp());

class LearningPlannerApp extends StatelessWidget {
  const LearningPlannerApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        title: 'Learning Planner',
        theme: ThemeData(colorSchemeSeed: Colors.teal, useMaterial3: true),
        home: const LearningDashboard(),
      );
}

class LearningGoal {
  LearningGoal({
    required this.id,
    required this.title,
    this.isComplete = false,
  });
  final int id;
  final String title;
  bool isComplete;
}

class LearningDashboard extends StatefulWidget {
  const LearningDashboard({super.key});

  @override
  State<LearningDashboard> createState() => _LearningDashboardState();
}

class _LearningDashboardState extends State<LearningDashboard> {
  final _controller = TextEditingController();
  final List<LearningGoal> _goals = [
    LearningGoal(id: 1, title: 'Flutter fundamentals', isComplete: true),
    LearningGoal(id: 2, title: 'Database design'),
    LearningGoal(id: 3, title: 'Build a portfolio project'),
  ];
  int _nextId = 4;

  int get _completed => _goals.where((goal) => goal.isComplete).length;

  Future<void> _addGoal() async {
    final title = _controller.text.trim();
    if (title.isEmpty) return;
    setState(() {
      _goals.add(LearningGoal(id: _nextId++, title: title));
      _controller.clear();
    });
  }

  void _toggle(LearningGoal goal, bool? value) {
    setState(() => goal.isComplete = value ?? false);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final progress = _goals.isEmpty ? 0.0 : _completed / _goals.length;
    return Scaffold(
      appBar: AppBar(title: const Text('Learning Planner')),
      body: SafeArea(
        child: Center(
          child: ConstrainedBox(
            constraints: const BoxConstraints(maxWidth: 640),
            child: Column(
              children: [
                Padding(
                  padding: const EdgeInsets.all(20),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(
                        'Progress: $_completed of ${_goals.length}',
                        style: Theme.of(context).textTheme.titleLarge,
                      ),
                      const SizedBox(height: 12),
                      LinearProgressIndicator(value: progress),
                    ],
                  ),
                ),
                Expanded(
                  child: _goals.isEmpty
                      ? const Center(child: Text('Add a goal to get started.'))
                      : ListView.builder(
                          itemCount: _goals.length,
                          itemBuilder: (context, index) {
                            final goal = _goals[index];
                            return CheckboxListTile(
                              key: ValueKey(goal.id),
                              title: Text(goal.title),
                              value: goal.isComplete,
                              onChanged: (value) => _toggle(goal, value),
                            );
                          },
                        ),
                ),
                Padding(
                  padding: const EdgeInsets.fromLTRB(16, 8, 16, 16),
                  child: Row(
                    children: [
                      Expanded(
                        child: TextField(
                          controller: _controller,
                          onSubmitted: (_) => _addGoal(),
                          decoration: const InputDecoration(
                            labelText: 'New learning goal',
                            border: OutlineInputBorder(),
                          ),
                        ),
                      ),
                      const SizedBox(width: 8),
                      FilledButton(
                        onPressed: _addGoal,
                        child: const Text('Add'),
                      ),
                    ],
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```

# Line-by-Line Explanation

- `LearningGoal` models a goal with a stable ID, a title, and mutable completion state for this small demo.
- `_goals` begins with three sample records; `_nextId` avoids using list positions as identity.
- `_completed` derives progress from the goal list, so the count cannot drift from the actual rows.
- `_addGoal` trims and validates the title, creates a new goal, and clears input inside `setState`.
- `_toggle` changes one goal and triggers a rebuild.
- `dispose` releases the text controller.
- The `progress` calculation handles an empty list to avoid division by zero.
- `ConstrainedBox` prevents excessively wide content; `Expanded` gives the list a bounded height.
- `LinearProgressIndicator` displays the derived ratio.
- `ListView.builder` creates rows as needed; `ValueKey` keeps row identity stable.

Removing the empty-list guard can produce invalid progress arithmetic. Removing `setState` leaves the visual progress and checkbox stale. Data disappears after restart because this version has no repository/database yet.

# Widget Tree

```text
LearningPlannerApp > MaterialApp > LearningDashboard > Scaffold
└── SafeArea > Center > ConstrainedBox > Column
    ├── Progress text + LinearProgressIndicator
    ├── Expanded > Empty state OR ListView.builder > CheckboxListTile
    └── Input row > TextField + Add button
```

# State Management Used

The sample uses local `StatefulWidget` state. The production path is to extract a repository and a controller/view model when persistence, multiple screens, or collaboration are introduced; do not add a state package solely for this one page.

# API/Database Used (if applicable)

None. This baseline is offline and in-memory. A next iteration can use SQLite for durable goals and a repository interface for testability. No remote service is necessary for personal goal tracking.

# Common Mistakes

- Storing progress separately from goals.
- Using list indices as goal identifiers.
- Forgetting the zero-goals progress case.
- Describing in-memory data as persistent.

# Improvements

- Add target dates, tags, and subtasks.
- Persist with a versioned database and test upgrade paths.
- Add search/filter and accessible progress descriptions.
- Split widgets and test derived progress as pure logic.

# Challenge Version

Turn this into a production-ready local-first planner: repository-backed persistence, edit/delete with undo, due-date filters, schema migration, tests, and accessible loading/error states.

# Summary

The capstone brings together typed records, validation, derived state, responsive layout, list identity, and an incremental architecture path from a runnable prototype to a maintainable app.

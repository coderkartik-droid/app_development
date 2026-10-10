# Project Overview

This roadmap turns the handbook's projects into a practical sequence. It is also a runnable Flutter checklist: learners can mark milestones complete and see progress. Completion is stored in memory for clarity; this app does not persist the roadmap.

# Features

- Ordered project milestones
- Completion toggle per milestone
- Progress indicator derived from checked items
- Suggested skill focus for each step
- Clear next-step guidance

# Folder Structure

```text
lib/
  main.dart
  models/milestone.dart
  data/roadmap.dart
  screens/roadmap_page.dart
```

# ASCII UI Preview

```text
+--------------------------+
| Flutter Project Roadmap  |
| 2 / 5 complete           |
| [████--------]           |
|--------------------------|
| [✓] Profile UI           |
| [✓] Calculator           |
| [ ] To-do + Notes        |
| [ ] Database project     |
| [ ] Capstone             |
+--------------------------+
```

# Step-by-Step Development Plan

1. Decide a progression from layout basics to state, persistence, and architecture.
2. Represent each step with a title and learning outcome.
3. Keep completion state local and derive progress.
4. Provide a responsive, readable checklist.
5. Add persistence only if learners need to retain their own roadmap state.

# Complete Runnable Code

Paste into `lib/main.dart`. No additional package is required.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const RoadmapApp());

class RoadmapApp extends StatelessWidget {
  const RoadmapApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        title: 'Project Roadmap',
        theme: ThemeData(colorSchemeSeed: Colors.blue, useMaterial3: true),
        home: const RoadmapPage(),
      );
}

class Milestone {
  Milestone({required this.title, required this.skill});
  final String title;
  final String skill;
  bool isComplete = false;
}

class RoadmapPage extends StatefulWidget {
  const RoadmapPage({super.key});

  @override
  State<RoadmapPage> createState() => _RoadmapPageState();
}

class _RoadmapPageState extends State<RoadmapPage> {
  final List<Milestone> _milestones = [
    Milestone(title: 'Profile UI', skill: 'Layouts, themes, reusable widgets'),
    Milestone(title: 'Calculator', skill: 'Input, validation, local state'),
    Milestone(title: 'To-do and Notes', skill: 'Lists, forms, model identity'),
    Milestone(title: 'Database project', skill: 'Schema, CRUD, migrations'),
    Milestone(title: 'Capstone', skill: 'Architecture, tests, polish'),
  ];

  int get _completed =>
      _milestones.where((milestone) => milestone.isComplete).length;

  @override
  Widget build(BuildContext context) {
    final progress =
        _milestones.isEmpty ? 0.0 : _completed / _milestones.length;
    return Scaffold(
      appBar: AppBar(title: const Text('Flutter Project Roadmap')),
      body: SafeArea(
        child: Column(
          children: [
            Padding(
              padding: const EdgeInsets.all(20),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text('$_completed of ${_milestones.length} complete'),
                  const SizedBox(height: 8),
                  LinearProgressIndicator(value: progress),
                  const SizedBox(height: 8),
                  const Text('Build one project at a time; test each milestone.'),
                ],
              ),
            ),
            Expanded(
              child: ListView.builder(
                itemCount: _milestones.length,
                itemBuilder: (context, index) {
                  final milestone = _milestones[index];
                  return CheckboxListTile(
                    value: milestone.isComplete,
                    title: Text(milestone.title),
                    subtitle: Text(milestone.skill),
                    onChanged: (value) => setState(
                      () => milestone.isComplete = value ?? false,
                    ),
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

# Line-by-Line Explanation

- `Milestone` pairs a project title with the main skills practiced and an in-memory completion flag.
- `_milestones` gives a deliberate learning order: interface, interaction, list/model work, database, then capstone integration.
- `_completed` derives its count from the checklist; no duplicate progress counter is kept.
- `progress` guards against division by zero if the roadmap is later loaded dynamically.
- The summary uses a `LinearProgressIndicator` and concise learning guidance.
- `Expanded` gives the list the remaining screen height.
- `ListView.builder` handles the milestone list efficiently.
- `CheckboxListTile` combines the label, skill summary, and completion control. `setState` causes progress and check state to rebuild together.

Removing the derived count can let progress disagree with checked rows. If list contents become dynamic, add stable milestone IDs and keys rather than relying on order.

# Widget Tree

```text
RoadmapApp > MaterialApp > RoadmapPage > Scaffold
└── SafeArea > Column
    ├── Progress summary + indicator + guidance
    └── Expanded > ListView.builder > CheckboxListTile
```

# State Management Used

The roadmap has a single local checklist, so `StatefulWidget` is sufficient. Durable progress would require a repository and explicit persistence state.

# API/Database Used (if applicable)

None. Milestones and completion flags are local sample data and reset when the app restarts.

# Common Mistakes

- Treating the roadmap as a race to finish rather than a learning sequence.
- Marking a project complete without testing or understanding it.
- Adding progress state that can disagree with milestone state.
- Assuming checkbox progress is persisted.

# Improvements

- Add links to relevant chapters and project briefs.
- Add estimated effort and self-assessment notes.
- Persist progress in a local database.
- Add a “next recommended step” based on incomplete milestones.

# Challenge Version

Add per-project acceptance criteria, test checklists, and notes. Persist progress and support exporting/importing the roadmap without losing data.

# Summary

The roadmap turns a large handbook into small deliverable projects, while the runnable checklist demonstrates derived progress and simple state updates. Treat each milestone as an opportunity to build, test, and explain—not merely copy code.

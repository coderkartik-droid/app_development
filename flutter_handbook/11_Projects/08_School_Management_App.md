# Project Overview

Build a simple classroom roster that shows students and lets a teacher mark attendance for the current demo session. The data is sample in-memory content; this project demonstrates lists, models, and state updates, not a production student-record system.

# Features

- Student roster with class labels
- Search by student name
- Mark present or absent
- Summary count of students marked present
- Empty search state

# Folder Structure

```text
lib/
  main.dart
  models/student.dart
  data/student_repository.dart
  screens/roster_page.dart
  widgets/student_tile.dart
```

# ASCII UI Preview

```text
+--------------------------+
| Class 7A         2 / 3   |
| [Search student...     ] |
|--------------------------|
| Amina Patel       [✓]    |
| Ben Carter        [ ]    |
| Chen Wu           [✓]    |
+--------------------------+
```

# Step-by-Step Development Plan

1. Define a student model with a stable ID and attendance state.
2. Store sample students in state.
3. Filter the visible list from the search query.
4. Toggle attendance by student ID.
5. Add privacy, authorization, and durable records before real deployment.

# Complete Runnable Code

Paste this into `lib/main.dart`. It requires no external packages and uses fictitious sample records only.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const SchoolApp());

class SchoolApp extends StatelessWidget {
  const SchoolApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        title: 'Class roster',
        theme: ThemeData(colorSchemeSeed: Colors.indigo, useMaterial3: true),
        home: const RosterPage(),
      );
}

class Student {
  Student({required this.id, required this.name, required this.className});
  final int id;
  final String name;
  final String className;
  bool present = false;
}

class RosterPage extends StatefulWidget {
  const RosterPage({super.key});

  @override
  State<RosterPage> createState() => _RosterPageState();
}

class _RosterPageState extends State<RosterPage> {
  final _searchController = TextEditingController();
  final List<Student> _students = [
    Student(id: 1, name: 'Amina Patel', className: '7A')..present = true,
    Student(id: 2, name: 'Ben Carter', className: '7A'),
    Student(id: 3, name: 'Chen Wu', className: '7A')..present = true,
  ];
  String _query = '';

  List<Student> get _visibleStudents => _students
      .where((student) => student.name.toLowerCase().contains(_query.toLowerCase()))
      .toList();

  int get _presentCount => _students.where((student) => student.present).length;

  @override
  void initState() {
    super.initState();
    _searchController.addListener(() {
      setState(() => _query = _searchController.text.trim());
    });
  }

  @override
  void dispose() {
    _searchController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(
          title: const Text('Class 7A'),
          actions: [
            Center(
              child: Padding(
                padding: const EdgeInsets.only(right: 16),
                child: Text('$_presentCount / ${_students.length} present'),
              ),
            ),
          ],
        ),
        body: Column(
          children: [
            Padding(
              padding: const EdgeInsets.all(16),
              child: TextField(
                controller: _searchController,
                decoration: const InputDecoration(
                  labelText: 'Search student',
                  prefixIcon: Icon(Icons.search),
                  border: OutlineInputBorder(),
                ),
              ),
            ),
            Expanded(
              child: _visibleStudents.isEmpty
                  ? const Center(child: Text('No students match this search.'))
                  : ListView(
                      children: _visibleStudents.map((student) {
                        return CheckboxListTile(
                          key: ValueKey(student.id),
                          title: Text(student.name),
                          subtitle: Text('Class ${student.className}'),
                          value: student.present,
                          onChanged: (value) => setState(
                            () => student.present = value ?? false,
                          ),
                        );
                      }).toList(),
                    ),
            ),
          ],
        ),
      );
}
```

# Line-by-Line Explanation

- `Student` contains stable identity, display name, class name, and attendance state.
- `_students` has fictitious sample data; initial attendance is set for two rows.
- `_query` stores the normalized text used to derive `_visibleStudents`.
- `_presentCount` derives the summary from source student state.
- The controller listener updates `_query` and triggers rebuilding as the teacher types.
- `dispose` releases the controller.
- The app bar displays the attendance count for the full roster, not just filtered rows.
- `Expanded` constrains the roster to the remaining height in the `Column`.
- `CheckboxListTile` updates one student's present flag inside `setState`.

Removing the controller listener makes search text changes invisible to the filter. If the summary were computed from only visible rows, searching would misleadingly change the class attendance total. Real attendance records must not be stored only in this sample state.

# Widget Tree

```text
SchoolApp > MaterialApp > RosterPage > Scaffold
├── AppBar > class label + attendance summary
└── Column
    ├── Search TextField
    └── Expanded > Empty result OR ListView > CheckboxListTile
```

# State Management Used

Local state is adequate for a single-session prototype. A real school app needs repository-backed attendance records and carefully scoped user/session state with auditability.

# API/Database Used (if applicable)

None. All names are fictional and the roster is in memory. Real student information is sensitive: define access controls, retention, audit, and applicable privacy requirements before deployment.

# Common Mistakes

- Using real student data in a demo or public repository.
- Computing attendance summary from only filtered results.
- Treating a checkbox as durable attendance evidence.
- Exposing private student details on a shared screen.

# Improvements

- Add date/session selection and attendance history.
- Persist changes in a database with migrations and audit timestamps.
- Add teacher authorization and data minimization.
- Provide accessible bulk actions and export controls.

# Challenge Version

Support multiple classes and attendance sessions. Persist each attendance mark with student ID, session ID, timestamp, and author; add tests for summary and history queries.

# Summary

This project teaches roster filtering, model identity, derived summaries, and checkbox state. It intentionally avoids real student data and production claims.

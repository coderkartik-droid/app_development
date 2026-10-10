# Introduction

A many-to-many relationship allows each record in one table to connect to many records in another, and vice versa. Students can enroll in many courses; each course can contain many students.

# Why this concept exists

Neither side can store one foreign key that represents every relationship. A junction table stores one row per association and can also hold relationship-specific fields such as enrollment date.

# Real-world analogy

A conference attendee can join many sessions, and each session has many attendees. A sign-in sheet for each attendee-session pair records each connection.

# Internal Working

Two one-to-many relationships form the many-to-many model: each junction row points to one record on each side.

```text
students 1 --- * enrollments * --- 1 courses
                 student_id FK
                 course_id FK
```

# Syntax

```sql
CREATE TABLE enrollments (
  student_id INTEGER NOT NULL REFERENCES students(id),
  course_id INTEGER NOT NULL REFERENCES courses(id),
  PRIMARY KEY (student_id, course_id)
);
```

# Complete Example 1

This runnable Flutter screen saves one student, two courses, and two enrollments, then joins names for display. Add `sqflite` and `path`.

```dart
import 'package:flutter/material.dart';
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<Database> openEnrollmentDb() async {
  final file = p.join(await getDatabasesPath(), 'many_to_many_demo.db');
  return openDatabase(file, version: 1, onConfigure: (db) async {
    await db.execute('PRAGMA foreign_keys = ON');
  }, onCreate: (db, version) async {
    await db.execute(
      'CREATE TABLE students (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL)',
    );
    await db.execute(
      'CREATE TABLE courses (id INTEGER PRIMARY KEY AUTOINCREMENT, title TEXT NOT NULL)',
    );
    await db.execute('''
      CREATE TABLE enrollments (
        student_id INTEGER NOT NULL REFERENCES students(id) ON DELETE CASCADE,
        course_id INTEGER NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
        PRIMARY KEY (student_id, course_id)
      )
    ''');
  });
}

void main() => runApp(const MaterialApp(home: EnrollmentPage()));

class EnrollmentPage extends StatefulWidget {
  const EnrollmentPage({super.key});
  @override
  State<EnrollmentPage> createState() => _EnrollmentPageState();
}

class _EnrollmentPageState extends State<EnrollmentPage> {
  late final Future<Database> _db = openEnrollmentDb();
  List<Map<String, Object?>> _rows = [];

  Future<void> _load() async {
    final db = await _db;
    await db.transaction((txn) async {
      final student = await txn.insert('students', {'name': 'Ari'});
      final flutter = await txn.insert('courses', {'title': 'Flutter'});
      final sql = await txn.insert('courses', {'title': 'SQL'});
      await txn.insert('enrollments', {'student_id': student, 'course_id': flutter});
      await txn.insert('enrollments', {'student_id': student, 'course_id': sql});
    });
    final result = await db.rawQuery('''
      SELECT students.name AS student, courses.title AS course
      FROM enrollments
      JOIN students ON students.id = enrollments.student_id
      JOIN courses ON courses.id = enrollments.course_id
      ORDER BY courses.title
    ''');
    if (mounted) setState(() => _rows = result);
  }

  @override
  void initState() {
    super.initState();
    _load();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Course enrollments')),
        body: ListView(
          children: _rows
              .map((row) => ListTile(
                    title: Text(row['course']! as String),
                    subtitle: Text('Student: ${row['student']}'),
                  ))
              .toList(),
        ),
      );
}
```

**Line-by-line guide:** Imports provide UI, path, and SQLite. `openEnrollmentDb` creates student, course, and junction tables and enables foreign-key checking. The composite primary key prevents duplicate enrollment pairs. `_load` transactionally creates related records using generated IDs, then joins three tables to return display names. The page updates only while mounted. `initState` loads; `build` renders one tile per association. Widget tree: `MaterialApp -> EnrollmentPage -> Scaffold -> AppBar + ListView/ListTile`.

**Database flow:** `students + courses -> enrollment pairs -> three-table JOIN -> rows -> UI`.

**Execution:** Ari appears in both courses. Removing the composite key permits duplicate enrollments; removing the junction table makes the relationship difficult to query and update safely.

```text
+----------------------+
| Course enrollments   |
|----------------------|
| Flutter              |
| Student: Ari         |
| SQL                  |
| Student: Ari         |
+----------------------+
```

**Common mistakes:** Duplicating lists of course IDs in a student row, omitting a uniqueness rule, or deleting a course while leaving orphan junction rows.

**Professional use case:** Product tags, users and roles, students and courses, or actors and films.

# Complete Example 2

Find all courses for one student:

```sql
SELECT courses.id, courses.title
FROM courses
JOIN enrollments ON enrollments.course_id = courses.id
WHERE enrollments.student_id = ?
ORDER BY courses.title;
```

The bound student ID filters the junction rows. Index `enrollments(student_id, course_id)` is already supported by the composite primary key in this order; a reverse query may need an index beginning with `course_id`.

# Complete Example 3

Store relationship metadata in the junction table:

```sql
ALTER TABLE enrollments ADD COLUMN enrolled_at INTEGER NOT NULL DEFAULT 0;
```

Enrollment date belongs to the student-course pair, not naturally to either parent. Add it via a versioned migration for a released database.

# Advanced Example

For symmetric lookup patterns, define two indexes: `(student_id, course_id)` and `(course_id, student_id)`. Use `EXPLAIN QUERY PLAN` to validate whether both are needed, since each index adds write and storage cost.

# Under the Hood

The junction table turns an unbounded set relationship into ordinary indexed rows. Composite primary keys ensure pair uniqueness and can support queries using the leftmost indexed column.

# Performance Notes

Index both traversal directions when they are common. Query through joins instead of issuing one query per associated item.

# Best Practices

- Use a junction table with foreign keys.
- Add a composite uniqueness/primary key for the pair.
- Store relationship attributes on the junction.
- Pick indexes for actual traversal directions.

# Common Mistakes

- Keeping comma-separated IDs in a field.
- Forgetting to prevent duplicate pairs.
- Assuming a composite index optimizes every column order.
- Leaving stale junction rows after parent deletion.

# Debugging

Inspect junction rows directly, run `PRAGMA foreign_key_check`, and test duplicate insertion. Compare join counts with expected fixture associations.

# Interview Questions

1. **How model many-to-many in SQL?** With a junction table and two foreign keys.
2. **What prevents duplicate associations?** A unique or composite primary key across both foreign keys.
3. **Where store relationship metadata?** On the junction row.
4. **Why may two indexes be needed?** Queries may traverse the relation from either side.

# Try It Yourself

Add a grade to student-course enrollment and query average grade per course.

# Challenge Project

Build a product-tag catalog with many-to-many filtering. Support adding/removing tags and ensure duplicate links cannot be created.

# Quick Revision Sheet

- Junction table represents each association.
- Pair key enforces uniqueness.
- Foreign keys protect referential integrity.
- Index for both access directions as measured.

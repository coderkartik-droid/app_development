# Project Overview

Build a compact notes app that lets a user create, view, and delete notes. This self-contained beginner implementation keeps notes in memory; it is structured so storage can later move behind a repository.

# Features

- Create notes from a title and body
- Display the newest note first
- Delete a note
- Validate empty input
- Show an empty state

# Folder Structure

```text
lib/
  main.dart
  models/note.dart
  data/notes_repository.dart
  screens/notes_page.dart
  screens/note_editor_page.dart
```

# ASCII UI Preview

```text
+--------------------------+
| Notes                    |
| [Search notes...       ] |
|--------------------------|
| Shopping                 |
| Milk, rice, apples... [x]|
| Meeting                  |
| Plan the sprint...    [x]|
|                    [+]   |
+--------------------------+
```

# Step-by-Step Development Plan

1. Create a `Note` model with stable ID, title, body, and creation time.
2. Add an editor dialog with validation.
3. Insert new notes into state and render with a lazy list.
4. Delete by ID and provide an empty state.
5. Replace in-memory storage with a database repository for a real app.

# Complete Runnable Code

Paste the code into `lib/main.dart`. It uses only Flutter's Material library.

```dart
import 'package:flutter/material.dart';

void main() => runApp(const NotesApp());

class NotesApp extends StatelessWidget {
  const NotesApp({super.key});

  @override
  Widget build(BuildContext context) => MaterialApp(
        title: 'Notes',
        theme: ThemeData(colorSchemeSeed: Colors.deepPurple, useMaterial3: true),
        home: const NotesPage(),
      );
}

class Note {
  Note({
    required this.id,
    required this.title,
    required this.body,
    required this.createdAt,
  });
  final int id;
  final String title;
  final String body;
  final DateTime createdAt;
}

class NotesPage extends StatefulWidget {
  const NotesPage({super.key});

  @override
  State<NotesPage> createState() => _NotesPageState();
}

class _NotesPageState extends State<NotesPage> {
  final List<Note> _notes = [];
  int _nextId = 1;

  Future<void> _createNote() async {
    final titleController = TextEditingController();
    final bodyController = TextEditingController();
    final noteData = await showDialog<(String, String)>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('New note'),
        content: SingleChildScrollView(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              TextField(
                controller: titleController,
                autofocus: true,
                decoration: const InputDecoration(labelText: 'Title'),
              ),
              TextField(
                controller: bodyController,
                maxLines: 4,
                decoration: const InputDecoration(labelText: 'Note'),
              ),
            ],
          ),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text('Cancel'),
          ),
          FilledButton(
            onPressed: () {
              final title = titleController.text.trim();
              if (title.isNotEmpty) {
                Navigator.pop(context, (title, bodyController.text.trim()));
              }
            },
            child: const Text('Save'),
          ),
        ],
      ),
    );
    titleController.dispose();
    bodyController.dispose();
    if (noteData == null) return;
    setState(() {
      _notes.insert(
        0,
        Note(
          id: _nextId++,
          title: noteData.$1,
          body: noteData.$2,
          createdAt: DateTime.now(),
        ),
      );
    });
  }

  void _deleteNote(int id) {
    setState(() => _notes.removeWhere((note) => note.id == id));
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Notes')),
        body: _notes.isEmpty
            ? const Center(child: Text('No notes yet. Tap + to write one.'))
            : ListView.builder(
                itemCount: _notes.length,
                itemBuilder: (context, index) {
                  final note = _notes[index];
                  return ListTile(
                    key: ValueKey(note.id),
                    title: Text(note.title, maxLines: 1, overflow: TextOverflow.ellipsis),
                    subtitle: Text(note.body, maxLines: 2, overflow: TextOverflow.ellipsis),
                    trailing: IconButton(
                      tooltip: 'Delete note',
                      onPressed: () => _deleteNote(note.id),
                      icon: const Icon(Icons.delete_outline),
                    ),
                  );
                },
              ),
        floatingActionButton: FloatingActionButton(
          onPressed: _createNote,
          child: const Icon(Icons.add),
        ),
      );
}
```

# Line-by-Line Explanation

- `Note` gives each record a stable ID, title, body, and timestamp.
- `_createNote` creates controllers for the temporary editor and opens an `AlertDialog`.
- The dialog returns a Dart record `(String, String)` only when the title is non-empty; cancel returns `null`.
- Controllers are disposed after the dialog closes, regardless of whether the user saved or cancelled.
- When data is returned, `setState` inserts a new note at the start with a unique ID.
- `_deleteNote` removes by stable ID, not by list position.
- `build` selects between an empty state and a lazily built list.
- Long text is capped with ellipsis so the list remains scannable.

Removing title validation permits blank records. Removing `setState` means the in-memory list changes but the rendered list does not update. This memory-only data disappears when the app process is closed.

# Widget Tree

```text
NotesApp > MaterialApp > NotesPage > Scaffold
├── AppBar
├── Empty-state text OR ListView.builder > ListTile
└── FloatingActionButton
    └── AlertDialog > title/body TextFields + Cancel/Save
```

# State Management Used

Local widget state is used for the short-lived prototype. Move note CRUD into a repository and use a provider/BLoC/etc. only when multiple screens need coordinated note state.

# API/Database Used (if applicable)

None in this runnable version. A production notes app should use a local database, version its schema, and consider export and recovery behavior.

# Common Mistakes

- Forgetting to dispose dialog controllers.
- Creating empty notes or using list indices as IDs.
- Showing full note bodies in a dense list.
- Claiming notes are saved when they exist only in memory.

# Improvements

- Add edit and search.
- Persist notes in SQLite or another suitable local database.
- Add created/updated timestamps and sorting.
- Add backup/export and deletion confirmation.

# Challenge Version

Implement search, edit, pinning, and durable storage. Add repository tests proving edits and deletes persist after reopening the database.

# Summary

This project combines data modeling, dialog input, validation, list rendering, and scoped deletion. Its main limitation—volatile memory—makes it a clear next step for database integration.

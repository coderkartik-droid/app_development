# Introduction

This project builds a local notes app with create, read, update, and delete actions. It uses a repository boundary and an in-memory implementation to keep the runnable UI dependency-free; replace the implementation with SQLite, Drift, or Hive for durable storage.

# Why this concept exists

Notes are user-authored data. They need reliable identity, editing, deletion, and an explicit decision about persistence, backup, and sync.

# Real-world analogy

A notebook needs pages with titles, stable order, edits, and a way to remove or archive a page. A list of strings is not enough once notes have metadata and lifecycle.

# Internal Working

The UI sends commands to a repository. The repository owns note records and returns updates. A database-backed version should persist each mutation, use stable IDs, and expose observable data or reload after changes.

# Syntax

```dart
class Note {
  const Note({required this.id, required this.title, required this.body});
  final String id;
  final String title;
  final String body;
}
```

# Complete Example 1

This complete runnable app implements a small notes CRUD interface in memory. Persistence is intentionally not claimed; the repository can be replaced with SQLite/Drift.

```dart
import 'package:flutter/material.dart';

class Note {
  const Note({required this.id, required this.title, required this.body});
  final String id;
  final String title;
  final String body;
}

class NotesRepository {
  final List<Note> _notes = [];
  List<Note> get all => List.unmodifiable(_notes);

  void save(Note note) {
    final index = _notes.indexWhere((item) => item.id == note.id);
    if (index < 0) {
      _notes.insert(0, note);
    } else {
      _notes[index] = note;
    }
  }

  void delete(String id) => _notes.removeWhere((note) => note.id == id);
}

void main() => runApp(const NotesApp());

class NotesApp extends StatelessWidget {
  const NotesApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: NotesPage());
}

class NotesPage extends StatefulWidget {
  const NotesPage({super.key});

  @override
  State<NotesPage> createState() => _NotesPageState();
}

class _NotesPageState extends State<NotesPage> {
  final _repository = NotesRepository();

  Future<void> _edit([Note? note]) async {
    final result = await Navigator.of(context).push<Note>(
      MaterialPageRoute(
        builder: (_) => NoteEditorPage(note: note),
      ),
    );
    if (!mounted) return;
    if (result != null) {
      setState(() => _repository.save(result));
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('My notes')),
        body: _repository.all.isEmpty
            ? const Center(child: Text('No notes yet'))
            : ListView.builder(
                itemCount: _repository.all.length,
                itemBuilder: (context, index) {
                  final note = _repository.all[index];
                  return ListTile(
                    title: Text(note.title),
                    subtitle: Text(note.body, maxLines: 1),
                    onTap: () => _edit(note),
                    trailing: IconButton(
                      icon: const Icon(Icons.delete_outline),
                      onPressed: () => setState(() => _repository.delete(note.id)),
                    ),
                  );
                },
              ),
        floatingActionButton: FloatingActionButton(
          onPressed: () => _edit(),
          child: const Icon(Icons.add),
        ),
      );
}

class NoteEditorPage extends StatefulWidget {
  const NoteEditorPage({required this.note, super.key});
  final Note? note;

  @override
  State<NoteEditorPage> createState() => _NoteEditorPageState();
}

class _NoteEditorPageState extends State<NoteEditorPage> {
  late final TextEditingController _title;
  late final TextEditingController _body;

  @override
  void initState() {
    super.initState();
    _title = TextEditingController(text: widget.note?.title ?? '');
    _body = TextEditingController(text: widget.note?.body ?? '');
  }

  void _save() {
    final title = _title.text.trim();
    if (title.isEmpty) return;
    final note = Note(
      id: widget.note?.id ?? DateTime.now().microsecondsSinceEpoch.toString(),
      title: title,
      body: _body.text.trim(),
    );
    Navigator.pop(context, note);
  }

  @override
  void dispose() {
    _title.dispose();
    _body.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: Text(widget.note == null ? 'New note' : 'Edit note')),
        body: Padding(
          padding: const EdgeInsets.all(20),
          child: Column(
            children: [
              TextField(controller: _title, decoration: const InputDecoration(labelText: 'Title')),
              TextField(
                controller: _body,
                decoration: const InputDecoration(labelText: 'Note'),
                maxLines: 6,
              ),
              FilledButton(onPressed: _save, child: const Text('Save')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `Note` models identity and editable content.
- `NotesRepository` returns an unmodifiable view and supports insert/replace/delete by ID.
- `_edit` pushes an editor route and receives a saved note as a route result.
- The page stores the returned object and rebuilds.
- `ListView.builder` renders notes on demand.
- The delete button removes by stable note ID, not visible list index.
- `NoteEditorPage` pre-fills fields when editing and starts empty when creating.
- `_save` validates title, reuses an existing ID or creates a demo ID, then returns the note through `Navigator.pop`.
- Both text controllers are disposed.

### Widget tree

```text
NotesPage → Scaffold → ListView/ListTile*
                     └→ FloatingActionButton
Editor route → Scaffold → Column(TextField title, TextField body, Save)
```

### Execution flow

```text
Add/edit → editor route → validate → return Note
                         |
                         v
                  repository save → list rebuild
Delete → repository delete by ID → list rebuild
```

### ASCII phone preview

```text
+---------------------------+
|          My notes         |
+---------------------------+
| Shopping list        [x]  |
| Milk, bread...            |
| Meeting notes        [x]  |
| Project plan...           |
|                       [+] |
+---------------------------+
```

### Example explanation

The repository uses stable IDs and separates storage behavior from presentation. This implementation is in memory, so restart loses notes. Replace methods with awaited database operations and expose a stream or reload after successful writes for persistence.

If the note ID is replaced during editing, it becomes a duplicate rather than an update. If controllers are not disposed, resources leak. If deletion is performed by list position, reordered data can delete the wrong note.

**Common mistakes:** Do not describe this fake repository as durable storage. Add confirmation/undo for destructive note deletion in production.

**Professional use case:** Personal notes, drafts, and meeting notes stored offline with a database-backed repository.

# Complete Example 2

Persist a note with Drift or SQLite using a stable ID and `updatedAt` timestamp. Parameterize all user content and update only the intended row.

# Complete Example 3

Add note search using a database query rather than loading an unbounded collection and filtering every record in the widget.

# Advanced Example

Add autosave with debouncing, draft recovery, conflict detection, export, attachments, and encryption if the threat model requires it. Define deletion and backup behavior before handling sensitive journals.

# Under the Hood

Route results return objects to the previous route. In production, repositories write database rows, and streams or queries notify the list. A domain ID ensures route edits update the original record.

# Performance Notes

- Use database pagination and indexed title search for large collections.
- Debounce autosave.
- Store attachments as files and note their paths.
- Avoid loading full note bodies for a preview list.

# Best Practices

- Use stable IDs and timestamps.
- Keep editor separate from repository.
- Validate titles and preserve drafts.
- Handle delete confirmation/undo.
- Test cold restart and migrations.

# Common Mistakes

- Claiming in-memory demo data persists.
- Deleting by list index.
- No empty/loading/error states.
- Saving every keystroke synchronously.
- Losing edits on route dismissal.

# Debugging

- Trace editor result and repository mutation.
- Verify ID remains unchanged during edit.
- Restart to test persistence.
- Check database row and migration.
- Test cancel without saving.

# Interview Questions

1. Why use a repository? It decouples the editor/list from storage.
2. Why stable IDs? To update/delete the intended note.
3. How to support large note collections? Indexed queries and pagination.
4. How to autosave safely? Debounce, persist drafts, and handle write failures.

# Try It Yourself

1. Add a search field.
2. Add a created/updated timestamp.
3. Add delete confirmation or undo.
4. Replace the fake repository with a database implementation.

# Challenge Project

Build a persistent notes app with search, autosave, database migration, delete undo, and an offline-first repository. Document whether notes are encrypted, backed up, or synced.

# Quick Revision Sheet

- Model notes with stable IDs.
- Separate UI and repository.
- Use a database for durable searchable records.
- Store attachments as files.
- Test edit, cancel, delete, restart, and migration.

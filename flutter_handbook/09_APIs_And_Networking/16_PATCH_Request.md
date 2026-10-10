# Introduction

PATCH applies a partial modification to a resource. Unlike a typical PUT replacement, a PATCH body usually describes only the fields or operations that should change.

# Why this concept exists

Updating one field should not require resending a large complete resource or accidentally overwriting fields changed elsewhere.

# Real-world analogy

PATCH is writing a correction beside one field on a form while leaving all other fields intact.

# Internal Working

The client sends PATCH to a resource URI with an API-defined patch document. Formats include a partial JSON object or standards such as JSON Merge Patch/JSON Patch. The server applies changes atomically according to its contract.

# Syntax

```dart
await http.patch(
  uri,
  headers: {'content-type': 'application/json'},
  body: jsonEncode({'completed': true}),
);
```

# Complete Example 1

This app patches a task's completion value against httpbin, which echoes requests and does not store tasks. Add `http`.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const PatchRequestApp());

class PatchRequestApp extends StatelessWidget {
  const PatchRequestApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: PatchRequestPage());
}

class PatchRequestPage extends StatefulWidget {
  const PatchRequestPage({super.key});

  @override
  State<PatchRequestPage> createState() => _PatchRequestPageState();
}

class _PatchRequestPageState extends State<PatchRequestPage> {
  bool _done = false;
  String _status = 'Not updated';

  Future<void> _toggle() async {
    final next = !_done;
    setState(() => _status = 'Updating...');
    try {
      final response = await http.patch(
        Uri.https('httpbin.org', '/anything/tasks/7'),
        headers: {'content-type': 'application/json'},
        body: jsonEncode({'done': next}),
      );
      if (!mounted) return;
      if (response.statusCode >= 200 && response.statusCode < 300) {
        setState(() {
          _done = next;
          _status = 'Updated';
        });
      } else {
        setState(() => _status = 'Update returned ${response.statusCode}');
      }
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Update failed');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('PATCH request')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              CheckboxListTile(
                title: const Text('Task complete'),
                value: _done,
                onChanged: (_) => _toggle(),
              ),
              Text(_status),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `_done` is the local state; `_status` communicates operation progress.
- `_toggle` computes the intended new value without immediately mutating state.
- PATCH sends only the `done` field to the task resource.
- A 2xx response commits the UI state; other statuses keep the previous state.
- Exceptions produce an explicit failure status.
- `CheckboxListTile` provides an accessible task toggle.

### Widget tree

```text
MaterialApp → PatchRequestPage → Scaffold → Center → Column
                                                  ├→ CheckboxListTile
                                                  └→ status
```

### Execution flow

```text
Tap -> compute desired done value -> PATCH field
    -> success? commit local state
    -> failure? keep old state and show failure
```

### ASCII UI preview

```text
+---------------------------+
|        PATCH request      |
+---------------------------+
| [x] Task complete         |
| Updated                   |
+---------------------------+
```

### Example explanation

This code uses a partial JSON body, but real patch media types differ. It delays changing `_done` until the server reports success, avoiding false confirmation. A production app may optimistically update but must roll back or mark pending on failure.

If the body contains a full object unintentionally, the endpoint may overwrite fields. If the UI state changes before request success without rollback, server and client can diverge.

**Common mistakes:** Do not assume every PATCH API uses a partial object with `application/json`.

# Complete Example 2

JSON Merge Patch may represent a partial object with a specific media type:

```dart
headers: {'content-type': 'application/merge-patch+json'}
```

Null may mean removal under that standard, so use the server's documented semantics.

# Complete Example 3

A revision/ETag can protect against overwriting concurrent changes:

```dart
headers: {'if-match': currentEtag}
```

Handle precondition failure by fetching the latest resource and resolving conflict.

# Advanced Example

Choose between partial-object semantics and operation-based patch formats. Validate allowed fields server-side so clients cannot update protected properties. Make each patch atomic and define retry behavior.

# Under the Hood

PATCH is not inherently idempotent in every patch format. Applying “increment by one” twice differs from setting `done=true` twice. Retry policies depend on operation semantics and server support.

# Performance Notes

- Send only changed fields.
- Avoid unnecessary round trips by batching compatible changes.
- Use optimistic updates only with reconciliation.

# Best Practices

- Follow exact patch format/content type.
- Restrict fields the client may change.
- Use concurrency validators for shared data.
- Handle partial failures and retries carefully.

# Common Mistakes

- Treating PATCH as universally idempotent.
- Sending wrong media type.
- Updating protected fields client-side.
- Optimistically changing UI with no rollback.

# Debugging

- Inspect patch body and content type.
- Verify field-level server permissions.
- Check ETag/precondition responses.
- Compare server representation before and after.

# Interview Questions

1. How does PATCH differ from PUT? It applies partial modifications rather than typically replacing the whole representation.
2. Is PATCH always idempotent? No, depends on operation semantics.
3. What is JSON Merge Patch? A standardized partial-object patch format.
4. How prevent lost updates? ETags/revisions and conditional requests.

# Try It Yourself

1. Add pending and rollback UI behavior.
2. Define null/removal semantics.
3. Add If-Match support.
4. Test applying the patch twice.

# Challenge Project

Build an editable task row that PATCHes one field, supports optimistic UI with rollback, and handles stale-resource conflicts.

# Quick Revision Sheet

- PATCH applies partial modifications.
- Patch body/media type is API-specific.
- Not all PATCH operations are idempotent.
- Use conditional requests for concurrency.
- Define optimistic rollback behavior.

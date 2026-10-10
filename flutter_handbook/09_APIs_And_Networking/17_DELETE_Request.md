# Introduction

DELETE requests that a server remove a resource identified by a URI. The result may be 200 with a body, 202 while deletion is pending, or 204 with no body.

# Why this concept exists

Clients need a standard way to remove records while keeping the target explicit and the operation auditable.

# Real-world analogy

DELETE is submitting a removal request for a specific file number. The receipt says whether it was removed, queued, or not found.

# Internal Working

The client sends DELETE to a resource URI. The server authenticates and authorizes, processes removal or tombstoning, and returns status. Local UI should reflect the result according to product policy; confirmation or undo may be appropriate for user data.

# Syntax

```dart
final response = await http.delete(resourceUri);
```

# Complete Example 1

This app demonstrates a delete request against an echo service. Add `http`; it does not delete a real record.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const DeleteRequestApp());

class DeleteRequestApp extends StatelessWidget {
  const DeleteRequestApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: DeleteRequestPage());
}

class DeleteRequestPage extends StatefulWidget {
  const DeleteRequestPage({super.key});

  @override
  State<DeleteRequestPage> createState() => _DeleteRequestPageState();
}

class _DeleteRequestPageState extends State<DeleteRequestPage> {
  bool _exists = true;
  String _status = 'Record exists';

  Future<void> _delete() async {
    final confirmed = await showDialog<bool>(
      context: context,
      builder: (context) => AlertDialog(
        title: const Text('Delete record?'),
        content: const Text('This demo sends a DELETE request.'),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Cancel'),
          ),
          FilledButton(
            onPressed: () => Navigator.pop(context, true),
            child: const Text('Delete'),
          ),
        ],
      ),
    );
    if (confirmed != true) return;
    try {
      final response = await http.delete(
        Uri.https('httpbin.org', '/anything/items/7'),
      );
      if (!mounted) return;
      final ok = response.statusCode >= 200 && response.statusCode < 300;
      setState(() {
        if (ok) _exists = false;
        _status = 'DELETE returned ${response.statusCode}';
      });
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Delete request failed');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('DELETE request')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_exists ? 'Sample record' : 'Record removed'),
              Text(_status),
              FilledButton(
                onPressed: _exists ? _delete : null,
                child: const Text('Delete'),
              ),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `_exists` represents the UI's record state.
- `_delete` asks for confirmation before sending a destructive request.
- The request targets a specific item URI.
- Any 2xx status is treated as accepted for this demo; production may distinguish 202, 204, and other results.
- UI removes the sample only on success.
- Errors leave the record visible and show failure.
- The button disables after removal.

### Widget tree

```text
Scaffold → Center → Column(record, status, Delete button)
Dialog → Cancel / Delete
```

### Execution flow

```text
Delete tap -> confirm -> DELETE /items/7
                         ├-> 2xx -> hide record
                         └-> error -> keep record and report
```

### ASCII UI preview

```text
+---------------------------+
|       DELETE request      |
+---------------------------+
| Sample record             |
| DELETE returned 200        |
| [          Delete        ] |
+---------------------------+
```

### Example explanation

The confirmation protects against accidental removal, and the state changes only after a successful response. An echo endpoint only tests transport and must not be mistaken for a real delete service.

If the UI removes the row before the request and has no rollback, failure causes the client to misrepresent server state. If a DELETE response is 204, do not attempt to parse a nonexistent JSON body.

**Common mistakes:** DELETE may be asynchronous or soft-delete; use server response semantics rather than assuming physical erasure.

# Complete Example 2

DELETE may return 204:

```dart
if (response.statusCode == 204) {
  // Treat as successful with no body.
}
```

# Complete Example 3

For an offline delete, retain a tombstone:

```text
mark record deleted locally + enqueue remote DELETE
```

Remove the tombstone only after server reconciliation so the record does not reappear.

# Advanced Example

For destructive account data, design confirmation, re-authentication, remote deletion, local cleanup, retries, and partial failure. Explain the scope to the user and comply with applicable retention obligations.

# Under the Hood

HTTP DELETE targets a URI and requests removal semantics. The server may implement soft deletion, asynchronous jobs, or idempotent repeat behavior. Client-side deletion and server data retention are separate concerns.

# Performance Notes

- Avoid repeatedly sending duplicate deletes.
- Queue offline deletes durably.
- Use undo where product policy permits.
- Keep deletion out of frame-critical work.

# Best Practices

- Target exact resource IDs.
- Confirm irreversible deletion.
- Handle 202/204 appropriately.
- Keep tombstones until sync completes.
- Clear related local files/caches.

# Common Mistakes

- Parsing a 204 response body.
- Removing UI state before failure handling.
- Assuming DELETE physically erases backups.
- Omitting account authorization.
- Losing offline delete intent.

# Debugging

- Verify URI, auth, and status.
- Check whether server soft-deletes or queues.
- Test repeated delete and not-found outcomes.
- Inspect tombstone and related file cleanup.

# Interview Questions

1. What does DELETE request? Removal of a resource at a URI.
2. Why might it return 204? Success with no response body.
3. What is a tombstone? Local delete intent retained for synchronization.
4. Does HTTP DELETE always erase all copies? No.

# Try It Yourself

1. Add undo before the server request.
2. Handle 202 accepted separately.
3. Add offline tombstone behavior.
4. Ensure 204 is not JSON-decoded.

# Challenge Project

Implement a delete flow for an offline note with confirmation, local tombstone, server retry, attachment cleanup, and explicit partial-failure state.

# Quick Revision Sheet

- DELETE targets one resource URI.
- Check response contract; 204 has no body.
- Confirm irreversible actions.
- Preserve offline deletion intent.
- Local delete is not necessarily complete erasure.

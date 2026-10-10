# Introduction

POST sends a representation to a server for processing, commonly to create a resource or trigger an operation. The server's response defines whether it was accepted, created, or rejected.

# Why this concept exists

Clients need to submit forms, create records, and initiate actions that are not simple reads. POST carries structured data in the request body.

# Real-world analogy

POST is submitting a completed application form to an office. The office validates it and returns a receipt or a list of problems.

# Internal Working

The client validates user input, serializes a body, sets content headers, and sends POST. The server validates again, applies authorization and business rules, stores or processes the request, then returns status and often a representation.

POST is not generally idempotent. Retrying after a timeout can create duplicates unless the API supports an idempotency key.

# Syntax

```dart
final response = await http.post(
  uri,
  headers: {'content-type': 'application/json'},
  body: jsonEncode({'title': title}),
);
```

# Complete Example 1

This app submits a demo post to httpbin, which echoes the request. Add `http`. The endpoint does not create a real product record.

```dart
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const PostRequestApp());

class PostRequestApp extends StatelessWidget {
  const PostRequestApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: PostRequestPage());
}

class PostRequestPage extends StatefulWidget {
  const PostRequestPage({super.key});

  @override
  State<PostRequestPage> createState() => _PostRequestPageState();
}

class _PostRequestPageState extends State<PostRequestPage> {
  final _title = TextEditingController();
  String _status = 'Ready';

  Future<void> _submit() async {
    final title = _title.text.trim();
    if (title.isEmpty) {
      setState(() => _status = 'Title is required');
      return;
    }
    setState(() => _status = 'Submitting...');
    try {
      final response = await http.post(
        Uri.https('httpbin.org', '/anything'),
        headers: {'content-type': 'application/json'},
        body: jsonEncode({'title': title}),
      );
      if (!mounted) return;
      setState(() => _status = 'Server responded ${response.statusCode}');
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Submission failed');
    }
  }

  @override
  void dispose() {
    _title.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('POST request')),
        body: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            children: [
              TextField(
                controller: _title,
                decoration: const InputDecoration(labelText: 'Title'),
              ),
              FilledButton(onPressed: _submit, child: const Text('Submit')),
              Text(_status),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `_title` captures user input; `_status` shows validation and request state.
- `_submit` trims and validates the title before networking.
- The request uses POST, a JSON content type, and an encoded JSON body.
- The result status is displayed; transport errors are handled separately.
- `dispose` releases the text controller.
- The form is built from a field, action, and feedback label.

### Widget tree

```text
MaterialApp → PostRequestPage → Scaffold → Padding → Column
                                                  ├→ TextField
                                                  ├→ Submit
                                                  └→ status
```

### Execution flow

```text
Input -> client validation -> jsonEncode -> POST body
     -> server validation/process -> HTTP response -> UI status
```

### ASCII UI preview

```text
+---------------------------+
|        POST request       |
+---------------------------+
| Title                     |
| [ New note             ]  |
| [        Submit        ]  |
| Server responded 200      |
+---------------------------+
```

### Example explanation

The content type tells the server to parse JSON. The demo endpoint echoes content; a production API might return 201 and a created resource ID.

If `jsonEncode` is removed, the body may not be valid JSON. If content type is removed, the server may parse it incorrectly. A timeout retry could duplicate a create, so use API-supported idempotency for safe retry.

**Common mistakes:** Do not assume HTTP 200 is the only valid POST success; check the endpoint contract.

**Professional use case:** Submit order, registration, feedback, or task creation payloads.

# Complete Example 2

Send an idempotency key for a create operation if the API supports it:

```dart
headers: {'Idempotency-Key': operationId}
```

Reuse the same key for retries of the same logical operation, not for unrelated creates.

# Complete Example 3

Decode a created resource:

```dart
if (response.statusCode == 201) {
  final created = jsonDecode(response.body) as Map<String, dynamic>;
}
```

Some APIs return an empty body or `Location`; follow the contract.

# Advanced Example

Disable duplicate submission while pending, but do not rely on UI disabling as the only duplicate guard. Persist a client operation ID for offline retries. Validate server errors into field-level messages where appropriate.

# Under the Hood

The client serializes the request body and headers. The server receives bytes, validates content type/schema, processes the command, and returns response metadata. The client must parse the body only when present and expected.

# Performance Notes

- Limit payload size.
- Avoid submitting on every keystroke.
- Use idempotency and prevent duplicate taps.
- Upload large content with streaming/multipart.

# Best Practices

- Validate on client and server.
- Set content type.
- Check status/body contract.
- Protect retryable creates with idempotency.
- Do not expose server secrets.

# Common Mistakes

- Missing JSON encoding/header.
- Retrying blindly.
- Treating client validation as server security.
- Ignoring 201/204 responses.

# Debugging

- Inspect sanitized body and content type.
- Verify server validation response.
- Check duplicate operation IDs.
- Distinguish timeout-after-commit from request-not-sent.

# Interview Questions

1. What is POST commonly used for? Create or process a submitted representation.
2. Is POST idempotent? Not generally.
3. How make retries safer? Use a server-supported idempotency key.
4. What is content type for? Identifies body format.

# Try It Yourself

1. Show server validation errors.
2. Add pending button state.
3. Mock a timeout after simulated server commit.
4. Add an idempotency key to a test request.

# Challenge Project

Build a create-order form with validation, duplicate-submit protection, idempotency key, typed response, and retry policy.

# Quick Revision Sheet

- POST submits a body for processing/creation.
- Encode JSON and set `Content-Type`.
- Check API-specific success statuses and response.
- POST retries can duplicate work; use idempotency.

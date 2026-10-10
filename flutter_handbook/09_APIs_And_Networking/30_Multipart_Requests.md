# Introduction

Multipart requests package several parts—text fields and files—into one HTTP body. `multipart/form-data` is common for form uploads.

# Why this concept exists

Files are binary data and may be sent alongside ordinary fields. Multipart provides boundaries and per-part metadata so the server can separate them.

# Real-world analogy

A parcel contains separate labeled packages: one envelope with a description and one box with a file. Boundaries show where each part starts and ends.

# Internal Working

The multipart client generates a boundary token, writes each field/file part with headers, then terminates the body with the boundary. The request's content type must include that same boundary; client libraries manage this when using their multipart request object.

# Syntax

```dart
final request = http.MultipartRequest('POST', uri)
  ..fields['caption'] = caption
  ..files.add(await http.MultipartFile.fromPath('image', path));
```

# Complete Example 1

This runnable UI sends a text-only multipart form to httpbin. Add `http`. It demonstrates form fields; file addition is covered below.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

void main() => runApp(const MultipartApp());

class MultipartApp extends StatelessWidget {
  const MultipartApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: MultipartPage());
}

class MultipartPage extends StatefulWidget {
  const MultipartPage({super.key});

  @override
  State<MultipartPage> createState() => _MultipartPageState();
}

class _MultipartPageState extends State<MultipartPage> {
  final _caption = TextEditingController(text: 'Travel photo');
  String _status = 'Ready';

  Future<void> _send() async {
    final request = http.MultipartRequest(
      'POST',
      Uri.https('httpbin.org', '/post'),
    )..fields['caption'] = _caption.text.trim();
    try {
      final response = await request.send();
      if (!mounted) return;
      setState(() => _status = 'Multipart response: ${response.statusCode}');
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Request failed');
    }
  }

  @override
  void dispose() {
    _caption.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Multipart form')),
        body: Padding(
          padding: const EdgeInsets.all(24),
          child: Column(
            children: [
              TextField(
                controller: _caption,
                decoration: const InputDecoration(labelText: 'Caption'),
              ),
              FilledButton(onPressed: _send, child: const Text('Send form')),
              Text(_status),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `MultipartRequest` creates a POST targeting the echo endpoint.
- `fields` adds a text part named `caption`.
- `send` submits the generated multipart body and returns a streamed response.
- The response status or transport error is displayed.
- The text controller is disposed.
- The UI contains a caption field and submit action.

### Widget tree

```text
MaterialApp → MultipartPage → Scaffold → Padding → Column
                                                  ├→ caption field
                                                  ├→ send button
                                                  └→ status
```

### Execution flow

```text
Field input -> MultipartRequest.fields -> boundary encoding -> POST
            -> echo server -> status -> UI
```

### ASCII UI preview

```text
+---------------------------+
|      Multipart form       |
+---------------------------+
| Caption                   |
| [ Travel photo         ]  |
| [      Send form       ]  |
| Multipart response: 200   |
+---------------------------+
```

### Example explanation

The request object manages multipart formatting. For a file, add a `MultipartFile` with a stable form field name. Do not manually set a mismatched boundary.

If fields are sent with ordinary JSON instead, a server expecting multipart will not parse them. If the request content type is overwritten, the boundary may be lost.

**Common mistakes:** Server field names must match exactly; `image`, `file`, and `upload` are not interchangeable.

# Complete Example 2

Add a file from a path:

```dart
request.files.add(
  await http.MultipartFile.fromPath('attachment', file.path),
);
```

This requires a real readable file and an endpoint that accepts the `attachment` part.

# Complete Example 3

Read streamed response content when needed:

```dart
final response = await request.send();
final body = await response.stream.bytesToString();
```

Avoid buffering enormous responses; parse or stream according to the API.

# Advanced Example

Multipart file APIs may support custom filename, MIME type, and byte stream. Validate actual content, size, and authorization server-side. For multiple files, define ordering and partial failure semantics.

# Under the Hood

Each multipart part includes Content-Disposition and possibly Content-Type headers. A unique boundary separates parts and is declared in the top-level Content-Type. The server parser uses the boundary to reconstruct fields/files.

# Performance Notes

- Stream files.
- Limit concurrent uploads and payload sizes.
- Avoid unnecessary base64 encoding, which increases payload size.
- Show upload progress where supported.

# Best Practices

- Use library-managed boundaries.
- Match server field names.
- Validate file and text fields.
- Handle streamed response and errors.
- Set reasonable size and timeout limits.

# Common Mistakes

- Manually setting boundary incorrectly.
- Base64-encoding files into JSON without need.
- Wrong field name or endpoint.
- Ignoring file size and content.

# Debugging

- Inspect request Content-Type boundary.
- Confirm multipart fields in server echo/test tool.
- Verify file exists and field name.
- Check server upload limits and accepted type.

# Interview Questions

1. What separates multipart sections? Boundary.
2. Why use `MultipartRequest`? It constructs correct multipart body/header.
3. How add a file? MultipartFile from path/bytes/stream.
4. Why avoid base64 for files? It expands data and adds encode/decode overhead.

# Try It Yourself

1. Add a file and a caption.
2. Upload two files with distinct field names.
3. Test an invalid field name.
4. Add client-side size validation.

# Challenge Project

Build a support-ticket multipart form with text, optional image attachment, progress, and robust validation/error states.

# Quick Revision Sheet

- Multipart carries fields/files with boundaries.
- Use MultipartRequest instead of manual encoding.
- Server field names and size rules matter.
- Stream large files; avoid needless base64.

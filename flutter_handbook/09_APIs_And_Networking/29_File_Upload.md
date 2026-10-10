# Introduction

Uploading sends a local file from the device to a server. Common use cases include profile photos, documents, and attachments.

# Why this concept exists

Users create or select content on-device, while a server may need to store, process, share, or back it up.

# Real-world analogy

Uploading is sending a signed document to an office. The sender must choose the right file, package it in an accepted format, and receive a receipt.

# Internal Working

The app obtains a file through a picker or app storage, validates size/type, and sends bytes using the API's required content type—often multipart/form-data. The server validates and stores it, then returns metadata. Large uploads need streaming, progress, retry/resume policy, and cancellation.

# Syntax

Multipart example requires package-specific file support:

```dart
request.files.add(await http.MultipartFile.fromPath('file', path));
```

# Complete Example 1

This complete snippet demonstrates multipart upload assuming the app already has an app-owned file path. Add `http`; a production UI also needs a file-picker package and platform configuration.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

Future<int> uploadFile(String path) async {
  final request = http.MultipartRequest(
    'POST',
    Uri.https('api.example.com', '/v1/uploads'),
  );
  request.files.add(await http.MultipartFile.fromPath('file', path));
  final streamed = await request.send().timeout(const Duration(minutes: 2));
  return streamed.statusCode;
}

void main() => runApp(const UploadApp());

class UploadApp extends StatelessWidget {
  const UploadApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: UploadPage());
}

class UploadPage extends StatefulWidget {
  const UploadPage({super.key});

  @override
  State<UploadPage> createState() => _UploadPageState();
}

class _UploadPageState extends State<UploadPage> {
  String _status = 'Choose an existing app file path in code to test';

  Future<void> _sendDemo() async {
    const path = '/path/to/app-owned-file.jpg';
    setState(() => _status = 'Uploading...');
    try {
      final code = await uploadFile(path);
      if (!mounted) return;
      setState(() => _status = 'Server status: $code');
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Upload failed; configure a real file path');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('File upload')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Text(_status, textAlign: TextAlign.center),
              FilledButton(onPressed: _sendDemo, child: const Text('Upload')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `uploadFile` builds a multipart POST request for the API's upload endpoint.
- `MultipartFile.fromPath` streams a file path into the form part.
- `request.send` returns a streamed response; timeout is set generously for file transfer.
- The page's placeholder path is intentionally not a valid universal path; connect a file picker or app-owned test file before running the upload.
- Status is presented to the user without claiming a successful upload for a failed path.

### Widget tree

```text
MaterialApp → UploadPage → Scaffold → Center → Column(status, Upload)
```

### Execution flow

```text
Select/prepare file -> validate -> multipart request -> server stores
                    <- status/metadata <- update UI
```

### ASCII UI preview

```text
+---------------------------+
|        File upload        |
+---------------------------+
| Upload complete           |
| [         Upload       ]  |
+---------------------------+
```

### Example explanation

The snippet is structurally complete but the example placeholder path must be replaced with a real app-owned file path and the endpoint with a trusted upload API. A production example should use a picker, validate the file, and show progress.

If the request content type is manually set to multipart without its boundary, the server may not parse the body. Let `MultipartRequest` generate it.

**Common mistakes:** Never trust the filename, extension, or MIME type supplied by a client as proof of safe content.

# Complete Example 2

Add metadata with the file:

```dart
request.fields['category'] = 'avatar';
request.files.add(await http.MultipartFile.fromPath('file', path));
```

The server must authorize and validate both the metadata and bytes.

# Complete Example 3

Use a pre-signed upload URL when the backend delegates large binary transfer to object storage. Treat the URL as a temporary secret and follow its method/headers/expiry contract.

# Advanced Example

For large uploads, stream with progress and cancellation; use chunked/resumable protocols if server-supported. Compute checksums where needed and avoid retrying from byte zero when a resumable session exists.

# Under the Hood

Multipart encodes fields and files with boundaries. The server parses parts, validates limits/content, and stores bytes. `MultipartRequest.send` returns a streamed response rather than an already-buffered body.

# Performance Notes

- Compress images before upload when quality permits.
- Stream files and show progress.
- Limit size and concurrent uploads.
- Upload in background only with platform-appropriate task support.

# Best Practices

- Validate size and content client-side for UX; revalidate server-side for security.
- Use HTTPS and authentication.
- Let multipart client set boundary headers.
- Clean temporary files.
- Support cancellation and retry/resume policy.

# Common Mistakes

- Invalid placeholder path.
- Incorrect multipart boundary.
- Trusting client MIME/extension.
- Buffering huge files.
- Sending upload credentials to arbitrary URLs.

# Debugging

- Verify file exists/read permissions.
- Inspect multipart field names and status.
- Check server size limit and accepted MIME.
- Test slow connection and interruption.
- Never log file content.

# Interview Questions

1. What is multipart/form-data? A body format for multiple form fields/files with boundaries.
2. Why not set boundary manually? Multipart client generates boundary to match body.
3. Why validate on server? Clients can be manipulated.
4. What are resumable uploads? Protocols that continue large transfers after interruption.

# Try It Yourself

1. Connect a file picker.
2. Add client size validation.
3. Add metadata field and inspect server response.
4. Add progress/cancel support.

# Challenge Project

Build a profile image upload flow with picker, preview, compression, multipart upload, progress, server validation, and cancellation.

# Quick Revision Sheet

- Use multipart for file + form fields.
- Generate boundaries with the request API.
- Validate file client- and server-side.
- Stream large files and expose progress.
- Use trusted destinations and HTTPS.

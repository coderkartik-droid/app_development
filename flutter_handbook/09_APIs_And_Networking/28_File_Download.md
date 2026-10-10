# Introduction

Downloading a file transfers bytes from a server to the device. The app may save them to a temporary cache, app documents directory, or a user-visible location depending on the feature.

# Why this concept exists

Images, reports, maps, and exports can be larger than ordinary JSON. They need byte handling, progress, storage location, and cleanup policy.

# Real-world analogy

Downloading is receiving a parcel. You need to verify it arrived intact, decide where to store it, and know whether it is temporary or something the user owns.

# Internal Working

A response body may be accessed as bytes or streamed. Large files should be streamed to disk rather than fully buffered in memory. Validate status, content length/checksum when available, sanitize filenames, and handle partial downloads and cancellation.

# Syntax

```dart
final response = await client.get(uri);
final bytes = response.bodyBytes;
await file.writeAsBytes(bytes);
```

This buffers the full response; use streaming for large files.

# Complete Example 1

This app downloads a small image into memory and displays it. Add `http`. For a large or durable file, use a streaming file API and platform storage location setup.

```dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:typed_data';

void main() => runApp(const DownloadApp());

class DownloadApp extends StatelessWidget {
  const DownloadApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: DownloadPage());
}

class DownloadPage extends StatefulWidget {
  const DownloadPage({super.key});

  @override
  State<DownloadPage> createState() => _DownloadPageState();
}

class _DownloadPageState extends State<DownloadPage> {
  Uint8List? _image;
  String _status = 'Tap download';

  Future<void> _download() async {
    setState(() => _status = 'Downloading...');
    try {
      final response = await http.get(
        Uri.https('picsum.photos', '/200'),
      ).timeout(const Duration(seconds: 15));
      if (response.statusCode != 200) {
        throw Exception('HTTP ${response.statusCode}');
      }
      if (!mounted) return;
      setState(() {
        _image = response.bodyBytes;
        _status = 'Image loaded (${response.bodyBytes.length} bytes)';
      });
    } catch (error) {
      if (!mounted) return;
      setState(() => _status = 'Download failed');
    }
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('Download image')),
        body: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              if (_image != null)
                Image.memory(_image!, width: 200, height: 200, fit: BoxFit.cover),
              Text(_status),
              FilledButton(onPressed: _download, child: const Text('Download')),
            ],
          ),
        ),
      );
}
```

### Line-by-line explanation

- `Uint8List` from `dart:typed_data` represents the downloaded binary bytes.
- `_download` starts progress, requests a sample image, and checks HTTP status.
- The bytes are stored in memory and passed to `Image.memory`.
- A timeout limits wait time; errors update status.
- The widget displays image only after success.

### Widget tree

```text
MaterialApp → DownloadPage → Scaffold → Center → Column
                                                  ├→ Image.memory (after load)
                                                  ├→ status
                                                  └→ Download button
```

### Execution flow

```text
Tap -> HTTP GET bytes -> status check -> memory bytes -> Image.memory
```

### ASCII UI preview

```text
+---------------------------+
|      Download image       |
+---------------------------+
|       [ image ]           |
| Image loaded (bytes)      |
| [       Download       ]  |
+---------------------------+
```

### Example explanation

This small-image demo buffers all bytes in memory. It is not suitable for large downloads or durable file storage. The app buffers the downloaded bytes only for a small image. 

If status validation is removed, an HTML error body may be passed to the image decoder. If the content is large, buffering can consume substantial memory.

**Common mistakes:** Treat filenames and content types from servers as untrusted.

# Complete Example 2

For durable storage, obtain an app documents/cache directory using an appropriate path-provider API, create an app-owned file, and write bytes or a stream. Add platform package dependencies as needed.

# Complete Example 3

Resume support requires server range support and careful validation of partial content; appending arbitrary bytes after a failed request can corrupt a file.

# Advanced Example

Use streaming, progress reporting, cancellation, checksum verification, atomic temp-file rename, disk-space checks, and cache eviction. For user-visible downloads, follow platform permissions and document-provider conventions.

# Under the Hood

HTTP response bytes arrive over the network. Buffering materializes them in memory; streaming applies backpressure and writes chunks. The operating system controls file paths, permissions, and available space.

# Performance Notes

- Stream large files.
- Avoid decoding large images repeatedly.
- Show progress and allow cancellation.
- Store temporary files in cache and clean them up.

# Best Practices

- Check status and content type.
- Sanitize filenames and paths.
- Verify integrity when required.
- Use temp file + atomic rename for durable downloads.
- Handle insufficient disk space and cancellation.

# Common Mistakes

- Buffering huge files.
- Trusting server filename/path.
- Writing partial content as complete.
- Omitting storage permission/location policy.
- Leaving temporary files forever.

# Debugging

- Check status, headers, content length, and bytes received.
- Verify target directory permissions.
- Test interrupted connection and low storage.
- Confirm downloaded file opens and checksum matches.

# Interview Questions

1. When use streaming? For large responses to avoid full memory buffering.
2. Why write to a temporary file first? Avoid presenting a partial download as complete.
3. What is backpressure? Controlling producer/consumer rate to avoid unbounded buffering.
4. Why sanitize filenames? Server-provided names are untrusted.

# Try It Yourself

1. Add an image size limit.
2. Show download progress.
3. Save to cache and delete on refresh.
4. Simulate interrupted download.

# Challenge Project

Build a report downloader with progress, cancellation, temporary file, validation, atomic move, and user-visible error handling.

# Quick Revision Sheet

- Small files: bytes may fit memory; large files: stream.
- Validate status, type, length, and integrity.
- Use safe app-owned locations and filenames.
- Handle cancellation, partial data, and disk space.
- Clean cache files.

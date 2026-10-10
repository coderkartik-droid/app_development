# Introduction

Server-Sent Events (SSE) is a text-based HTTP mechanism for streaming events from a server to a client. It is one-way: the server sends updates; the client sends commands with ordinary HTTP requests.

# Why this concept exists

Some apps need live server updates but do not need a bidirectional socket. SSE can be simpler for notifications, progress updates, and monitoring feeds.

# Real-world analogy

SSE is a radio broadcast: the station sends updates to listeners, while listeners use a separate phone line if they need to respond.

# Internal Working

The client makes a long-lived HTTP request with `Accept: text/event-stream`. The server keeps the response open and writes events separated by lines and blank lines. Events may include `id`, `event`, and `data` fields. Reconnection and last-event resume behavior depend on client/server support.

# Syntax

```text
id: 42
event: update
data: {"status":"ready"}

```

# Complete Example 1

This runnable UI demonstrates consuming a simple SSE-like stream from a fake Dart stream, avoiding dependency on a public SSE server. A real app needs an SSE-capable HTTP client/parser.

```dart
import 'dart:async';
import 'package:flutter/material.dart';

class FakeEventService {
  Stream<String> events() async* {
    for (var i = 1; i <= 5; i++) {
      await Future<void>.delayed(const Duration(seconds: 1));
      yield 'Update $i';
    }
  }
}

void main() => runApp(const SseApp());

class SseApp extends StatelessWidget {
  const SseApp({super.key});

  @override
  Widget build(BuildContext context) =>
      const MaterialApp(home: SsePage());
}

class SsePage extends StatefulWidget {
  const SsePage({super.key});

  @override
  State<SsePage> createState() => _SsePageState();
}

class _SsePageState extends State<SsePage> {
  final _service = FakeEventService();
  StreamSubscription<String>? _subscription;
  final List<String> _events = [];

  @override
  void initState() {
    super.initState();
    _subscription = _service.events().listen((event) {
      if (!mounted) return;
      setState(() => _events.insert(0, event));
    });
  }

  @override
  void dispose() {
    _subscription?.cancel();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => Scaffold(
        appBar: AppBar(title: const Text('SSE event stream')),
        body: _events.isEmpty
            ? const Center(child: Text('Waiting for events...'))
            : ListView(
                children: _events.map((event) => ListTile(title: Text(event))).toList(),
              ),
      );
}
```

### Line-by-line explanation

- `FakeEventService.events` yields a new event once per second, simulating a server stream.
- `_subscription` owns the stream listener.
- `initState` subscribes once and inserts each event into the visible list.
- `mounted` guards updates if an event arrives near disposal.
- `dispose` cancels the subscription.
- The screen shows a waiting message or most recent events.

### Widget tree

```text
MaterialApp → SsePage → Scaffold
                      ├→ waiting state
                      └→ ListView of events
```

### Execution flow

```text
Fake server stream yields event -> subscription callback
                                 -> update event list -> rebuild UI
Page disposed -> cancel subscription
```

### ASCII UI preview

```text
+---------------------------+
|     SSE event stream      |
+---------------------------+
| Update 5                  |
| Update 4                  |
| Update 3                  |
+---------------------------+
```

### Example explanation

The example teaches the stream lifecycle but does not implement the SSE wire protocol. A production parser must handle `text/event-stream`, multiline data, event IDs, keep-alive comments, reconnect, and cancellation.

If subscription cancellation is removed, the stream can continue to update a page that is no longer visible. If event parsing assumes one line per event, multiline `data:` records can be corrupted.

**Common mistakes:** SSE is server-to-client only; send client actions through ordinary HTTP endpoints.

# Complete Example 2

Send a command with HTTP and receive updates through SSE:

```text
POST /jobs -> returns jobId
GET /jobs/{jobId}/events (SSE) -> progress events
```

Keep command and event channels logically correlated by job ID.

# Complete Example 3

Use event IDs to resume:

```text
Last-Event-ID: 42
```

Resume only if server guarantees event retention and ordering semantics.

# Advanced Example

Select SSE when the platform, proxy, and backend support long-lived HTTP streams. Define reconnect behavior, event retention, heartbeat, auth, stream limits, and background lifecycle. On web, browser APIs and CORS influence available choices.

# Under the Hood

SSE is a text stream over HTTP, not a protocol upgrade. The server flushes event frames on a long-lived response. The client parser groups lines into events and emits them to a stream consumer.

# Performance Notes

- Avoid reconnect storms.
- Parse incrementally rather than buffer the whole stream.
- Limit event payload size.
- Stop streams when no longer useful.

# Best Practices

- Use authenticated HTTPS streams.
- Handle multiline data and event IDs.
- Reconnect with backoff.
- Reconcile missed events.
- Cancel subscriptions and HTTP stream.

# Common Mistakes

- Treating SSE as bidirectional.
- Parsing each network chunk as a full event.
- Ignoring reconnect and Last-Event-ID.
- Buffering indefinitely.
- Forgetting CORS/platform constraints.

# Debugging

- Verify `Accept: text/event-stream` and response content type.
- Inspect raw frames and blank-line boundaries.
- Test slow chunking and connection close.
- Check heartbeat/proxy timeout behavior.

# Interview Questions

1. Is SSE bidirectional? No, server-to-client.
2. How does it differ from WebSocket? SSE stays HTTP text stream; WebSocket is bidirectional protocol upgrade.
3. What is Last-Event-ID? Resume cursor for event streams where supported.
4. Why not split every chunk as an event? Chunk boundaries do not necessarily align with event boundaries.

# Try It Yourself

1. Parse sample `id/event/data` frames.
2. Add cancellation.
3. Design event resume and duplicate handling.
4. Compare SSE with polling for progress updates.

# Challenge Project

Build a job-progress screen using an SSE service abstraction, typed event parser, reconnect/backoff, event cursor, and HTTP command endpoint.

# Quick Revision Sheet

- SSE is server-to-client streaming over HTTP.
- `text/event-stream`; event frames separated by blank lines.
- Network chunks may split frames.
- Use event IDs for resume if supported.
- Reconnect, parse incrementally, and cancel.
